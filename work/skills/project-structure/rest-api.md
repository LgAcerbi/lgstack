# REST API layout (Express)

The default for new REST APIs. In an existing project, the project's own conventions win (see `SKILL.md`).

The examples target Express 5 and zod 3, and they type-check with `strict`, `noUncheckedIndexedAccess` and `exactOptionalPropertyTypes` on.

## Folders

```
src/
  routes/         wiring only: path, method, middleware
  controllers/    HTTP in and out, parse with schemas, map results to status codes
  services/       business logic, typed results
  repositories/   interfaces + implementations, data access only
  models/         domain types, brands and their parsers
  schemas/        zod schemas for the API boundary
  middlewares/    error handler, logging, auth
  config/         env parsed with zod once, exported typed
  lib/            small shared helpers with no domain knowledge
  container.ts    composition root: creates and wires every class
  app.ts          createApp(deps): builds the Express app
  server.ts       loads config, builds the container, calls listen
```

Files are named `<feature>.<layer>.ts`. Tests sit next to the file they test as `<feature>.<layer>.test.ts`.

## Who may import what

| Layer | May import | Must not |
| --- | --- | --- |
| `routes/` | its controller (as a parameter), `middlewares/` | services, repositories |
| `controllers/` | services, `schemas/`, `models/`, `lib/` | repositories, `config/` |
| `services/` | repository interfaces, other services, `models/`, `lib/` | Express, `req`/`res`, `schemas/`, HTTP status codes |
| `repositories/` | `models/`, `lib/` | services, controllers, Express |
| `models/` | `lib/` | anything else |
| `schemas/` | `models/` | services, repositories |
| `lib/` | nothing from the app | any layer |
| `config/` | zod | any layer |

Environment variables are read only in `config/`. `process.env` appears nowhere else.

## Responsibilities

- **Routes**: path, method and middleware. No logic.
- **Controllers**: the only layer that knows `req` and `res`. Parse input with schemas, call one service method, map the result to a status code and body.
- **Services**: business rules. Receive typed input (never raw request data), return typed results for expected failures, throw only for bugs.
- **Repositories**: data access behind an interface. No business rules.
- **Models**: domain types the whole app shares, including brands like `UserId` and their parsers.
- **Schemas**: request and response shapes for the API. The domain does not depend on them.

## Classes and wiring

- Services, controllers and repositories are classes with constructor injection.
- Repositories are an interface plus one or more implementations. Services depend on the interface only. Services and controllers get no interface: one implementation doesn't need one.
- Controller handlers are arrow-function properties so `this` survives being passed to Express.
- Everything is created by hand in `container.ts`. No DI library, no decorators.

## Example: one feature end to end

### `models/user.model.ts`

```ts
export type UserId = number & { readonly __brand: "UserId" };

export function parseUserId(raw: string): UserId | undefined {
  if (!/^[1-9]\d*$/.test(raw)) return undefined;
  const n = Number(raw);
  return Number.isSafeInteger(n) ? (n as UserId) : undefined;
}

export type Role = "admin" | "member";

export type User = {
  readonly id: UserId;
  readonly name: string;
  readonly email: string;
  readonly role: Role;
  readonly createdAt: Date;
};
```

### `schemas/user.schema.ts`

```ts
import { z } from "zod";

export const CreateUserSchema = z.object({
  name: z.string().trim().min(1),
  email: z.string().email(),
  role: z.enum(["admin", "member"]).default("member"),
});

export type CreateUserInput = z.infer<typeof CreateUserSchema>;
```

### `repositories/user.repository.ts`

```ts
import type { User, UserId } from "../models/user.model";

export interface UserRepository {
  findAll(): Promise<readonly User[]>;
  findById(id: UserId): Promise<User | undefined>;
  insert(data: Omit<User, "id">): Promise<User>;
  delete(id: UserId): Promise<boolean>;
}

export class InMemoryUserRepository implements UserRepository {
  private users: User[] = [];
  private lastId = 0;

  constructor(seed: readonly User[] = []) {
    this.users = [...seed];
    this.lastId = Math.max(0, ...seed.map((u) => u.id));
  }

  async findAll() {
    return this.users.toSorted((a, b) => a.name.localeCompare(b.name));
  }

  async findById(id: UserId) {
    return this.users.find((u) => u.id === id);
  }

  async insert(data: Omit<User, "id">) {
    const user: User = { ...data, id: ++this.lastId as UserId };
    this.users.push(user);
    return user;
  }

  async delete(id: UserId) {
    const index = this.users.findIndex((u) => u.id === id);
    if (index === -1) return false;
    this.users.splice(index, 1);
    return true;
  }
}
```

Repository methods are `async` even in memory, so swapping in a real database doesn't change any signature.

### `services/user.service.ts`

```ts
import type { CreateUserInput } from "../schemas/user.schema";
import type { User, UserId } from "../models/user.model";
import type { UserRepository } from "../repositories/user.repository";

export type CreateUserResult = { ok: true; user: User } | { ok: false; error: "email_taken" };
export type DeleteUserResult = { ok: true } | { ok: false; error: "not_found" };

export class UserService {
  constructor(private readonly users: UserRepository) {}

  list() {
    return this.users.findAll();
  }

  get(id: UserId) {
    return this.users.findById(id);
  }

  async create(input: CreateUserInput): Promise<CreateUserResult> {
    const all = await this.users.findAll();
    if (all.some((u) => u.email === input.email)) return { ok: false, error: "email_taken" };
    const user = await this.users.insert({ ...input, createdAt: new Date() });
    return { ok: true, user };
  }

  async delete(id: UserId): Promise<DeleteUserResult> {
    return (await this.users.delete(id)) ? { ok: true } : { ok: false, error: "not_found" };
  }
}
```

The service imports only the inferred input *type* from `schemas/`, never the schema itself, so it stays independent of how input is parsed.

### `controllers/user.controller.ts`

```ts
import type { Request, Response } from "express";
import { parseUserId } from "../models/user.model";
import { CreateUserSchema } from "../schemas/user.schema";
import type { UserService } from "../services/user.service";

export class UserController {
  constructor(private readonly userService: UserService) {}

  list = async (_req: Request, res: Response) => {
    res.json(await this.userService.list());
  };

  get = async (req: Request<{ id: string }>, res: Response) => {
    const id = parseUserId(req.params.id);
    if (!id) return void res.status(404).json({ error: "not_found" });
    const user = await this.userService.get(id);
    if (!user) return void res.status(404).json({ error: "not_found" });
    res.json(user);
  };

  create = async (req: Request, res: Response) => {
    const parsed = CreateUserSchema.safeParse(req.body);
    if (!parsed.success) return void res.status(400).json({ error: "invalid_body", issues: parsed.error.issues });
    const result = await this.userService.create(parsed.data);
    if (!result.ok) return void res.status(409).json({ error: result.error });
    res.status(201).json(result.user);
  };

  delete = async (req: Request<{ id: string }>, res: Response) => {
    const id = parseUserId(req.params.id);
    if (!id) return void res.status(404).json({ error: "not_found" });
    const result = await this.userService.delete(id);
    if (!result.ok) return void res.status(404).json({ error: result.error });
    res.status(204).send();
  };
}
```

### `routes/user.routes.ts`

```ts
import { Router } from "express";
import type { UserController } from "../controllers/user.controller";

export function userRoutes(controller: UserController): Router {
  return Router()
    .get("/", controller.list)
    .get("/:id", controller.get)
    .post("/", controller.create)
    .delete("/:id", controller.delete);
}
```

### `config/env.ts`

```ts
import { z } from "zod";

const EnvSchema = z.object({
  PORT: z.coerce.number().int().positive().default(3000),
  NODE_ENV: z.enum(["development", "test", "production"]).default("development"),
});

export type Env = z.infer<typeof EnvSchema>;

export function loadEnv(source: NodeJS.ProcessEnv = process.env): Env {
  return EnvSchema.parse(source);
}
```

`z.coerce` is acceptable here because the schema still rejects non-numeric and out-of-range values.

### `container.ts`

```ts
import { UserController } from "./controllers/user.controller";
import { InMemoryUserRepository, type UserRepository } from "./repositories/user.repository";
import { UserService } from "./services/user.service";

export type Overrides = { userRepository?: UserRepository };

export function createContainer(overrides: Overrides = {}) {
  const userRepository = overrides.userRepository ?? new InMemoryUserRepository();
  const userService = new UserService(userRepository);
  const userController = new UserController(userService);
  return { userController };
}

export type Container = ReturnType<typeof createContainer>;
```

### `app.ts`

```ts
import express from "express";
import type { Container } from "./container";
import { errorHandler } from "./middlewares/error.middleware";
import { userRoutes } from "./routes/user.routes";

export function createApp(container: Container) {
  const app = express();
  app.use(express.json());
  app.use("/users", userRoutes(container.userController));
  app.use(errorHandler);
  return app;
}
```

### `server.ts`

```ts
import { createApp } from "./app";
import { loadEnv } from "./config/env";
import { createContainer } from "./container";

const env = loadEnv();
createApp(createContainer()).listen(env.PORT);
```

### `middlewares/error.middleware.ts`

```ts
import type { ErrorRequestHandler } from "express";

export const errorHandler: ErrorRequestHandler = (err, _req, res, _next) => {
  console.error(err); // replace with the project's logger
  res.status(500).json({ error: "internal_error" });
};
```

On Express 4, rejected promises from `async` handlers don't reach this middleware. Use Express 5, or wrap handlers so rejections are passed to `next`.

### `controllers/user.controller.test.ts`

```ts
import request from "supertest";
import { beforeEach, describe, expect, it } from "vitest";
import { createApp } from "../app";
import { createContainer } from "../container";
import { InMemoryUserRepository } from "../repositories/user.repository";

describe("users", () => {
  let app: ReturnType<typeof createApp>;

  beforeEach(() => {
    app = createApp(createContainer({ userRepository: new InMemoryUserRepository() }));
  });

  it("creates a user that can then be fetched", async () => {
    const created = await request(app).post("/users").send({ name: "Dan", email: "dan@example.com" });
    expect(created.status).toBe(201);

    const fetched = await request(app).get(`/users/${created.body.id}`);
    expect(fetched.body.name).toBe("Dan");
  });

  it("returns 404 when deleting a user that does not exist", async () => {
    expect((await request(app).delete("/users/999")).status).toBe(404);
  });
});
```

Each test builds a fresh app with its own repository, so tests never share state.

## Adding a feature

For a new feature `orders`, add one file per layer that needs it: `models/order.model.ts`, `schemas/order.schema.ts`, `repositories/order.repository.ts`, `services/order.service.ts`, `controllers/order.controller.ts`, `routes/order.routes.ts`. Then wire the classes in `container.ts` and mount the routes in `app.ts`. If `OrderService` needs users, inject `UserService`, never `UserRepository`.
