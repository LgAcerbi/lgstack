# TypeScript patterns

Examples for the rules in `SKILL.md`.

## Discriminated unions

```ts
// Don't: contradictory states can exist (loading with an error?)
type FetchState = { loading: boolean; data?: User; error?: string };

// Do: only valid states exist
type FetchState =
  | { kind: "loading" }
  | { kind: "ready"; data: User }
  | { kind: "error"; error: string };
```

Pick one discriminant name and use it everywhere.

## Branded types

```ts
type UserId = string & { readonly __brand: "UserId" };

function isUserId(input: string): input is UserId {
  return /^[0-9a-f-]{36}$/.test(input);
}

function parseUserId(input: string): UserId {
  if (!isUserId(input)) throw new Error(`Invalid user id: ${input}`);
  return input;
}
```

Validate once at the boundary. Code downstream trusts the type.

## Constructive modeling

```ts
type NonEmpty<T> = [T, ...T[]];

// Don't: every caller must remember the length check
function pickWinner(entries: string[]): string {
  if (entries.length === 0) throw new Error("no entries");
  return entries[Math.floor(Math.random() * entries.length)];
}

// Do: an empty value cannot exist
function pickWinner(entries: NonEmpty<string>): string {
  return entries[Math.floor(Math.random() * entries.length)];
}

const isNonEmpty = <T>(arr: T[]): arr is NonEmpty<T> => arr.length > 0;
```

```ts
// Don't: a comment holds the invariant
type TimeRange = { start: Date; end: Date }; // start <= end

// Do: a negative range cannot be written
type TimeRange = { start: Date; durationMs: number };
```

## Simplest total type

```ts
// Fine: [] sums to 0, every input is handled
const sum = (xs: number[]) => xs.reduce((a, b) => a + b, 0);

// Don't: partiality smuggled past the compiler
function newest(sessions: Session[]): Session {
  return sessions.at(0)!;
}

// Do: strengthen the input, or return Session | undefined
function newest(sessions: NonEmpty<Session>): Session {
  return sessions[0];
}
```

## Schemas at boundaries

```ts
import { z } from "zod";

const UserSchema = z.object({
  id: z.string().uuid(),
  role: z.enum(["admin", "member"]),
});
type User = z.infer<typeof UserSchema>;

// Don't
const user = (await res.json()) as User;

// Do
const user = UserSchema.parse(await res.json());
```

Use `safeParse` when failure is an expected branch. When the type is written first, annotate the schema with it (`const s: z.ZodType<User> = ...`) so the compiler rejects a schema that proves less than the type.

## Parse, don't coerce

Route params, query strings and headers are always strings, and they come from outside.

```ts
// Don't: Number() accepts "1.0", "0x1", "1e0", " 1" and "" (as 0)
const id = Number(req.params.id);

// Do: one accepted format, everything else is rejected
type UserId = number & { readonly __brand: "UserId" };

function parseUserId(raw: string): UserId | undefined {
  if (!/^[1-9]\d*$/.test(raw)) return undefined;
  const n = Number(raw);
  return Number.isSafeInteger(n) ? (n as UserId) : undefined;
}

const id = parseUserId(req.params.id);
if (id === undefined) return res.status(404).json({ error: "User not found" });
```

The one `as` is allowed because it comes right after the check that proves it. With a schema library, the same thing is `z.string().regex(/^[1-9]\d*$/).transform(Number)`.

Then type the domain field to match (`id: UserId` on `User`, not `id: any`), so a raw string can't reach a lookup anywhere.

## `satisfies` over `as`

```ts
const routes = {
  home: "/",
  user: "/users/:id",
} satisfies Record<string, `/${string}`>;
// routes.home keeps its literal type "/"
```

## Exhaustiveness

```ts
function label(state: FetchState): string {
  switch (state.kind) {
    case "loading": return "Loading…";
    case "ready": return state.data.name;
    case "error": return state.error;
    default: {
      const _exhaustive: never = state;
      return _exhaustive;
    }
  }
}
```

## Object arguments

```ts
// Don't: what do true and 3 mean?
createInvite("a@b.com", true, 3);

// Do
createInvite({ email: "a@b.com", isAdmin: true, expiresInDays: 3 });
```

## Literal unions over `enum`

```ts
// Don't: emits runtime code, numeric enums accept any number
enum Role { Admin, Member }

// Do: a union of literals
type Role = "admin" | "member";

// Do: when you also need the values at runtime
const ROLES = ["admin", "member"] as const;
type Role = (typeof ROLES)[number];
const RoleSchema = z.enum(ROLES);
```

## `undefined` for absence

```ts
// Don't: two ways to say "no avatar"
type User = { avatarUrl?: string | null };

// Do: one way inside the app
type User = { avatarUrl?: string };

// Convert at the boundary when the database or API uses null
const user: User = { ...row, avatarUrl: row.avatar_url ?? undefined };
```

## Narrowing order

Use the most precise narrowing the types allow.

```ts
type Event =
  | { kind: "click"; x: number; y: number }
  | { kind: "key"; key: string };

// 1. Discriminant: the compiler narrows each branch
function describe(e: Event) {
  switch (e.kind) {
    case "click": return `click at ${e.x},${e.y}`;
    case "key": return `key ${e.key}`;
  }
}

// 2. `in`, when there is no discriminant
if ("key" in e) e.key;

// 3. `typeof` / `instanceof`, for primitives and classes
if (typeof value === "string") value.toUpperCase();
if (err instanceof HttpError) err.status;

// 4. A user-defined guard, only when 1 to 3 can't express it
// 5. `as`, only right after a check that proves it
```

## Honest type guards

```ts
// Don't: the name promises a User, the body checks one field
function isUser(x: unknown): x is User {
  return typeof x === "object" && x !== null && "id" in x;
}

// Do: let the schema do the full check
function isUser(x: unknown): x is User {
  return UserSchema.safeParse(x).success;
}
```

A guard is only as safe as its body. If writing a complete check by hand is long, that is the signal to use a schema.

## Types at the root

```ts
// Symptom fix: the call site converts, but the type still allows the bug
type User = { id: any; name: string };
findUser(Number(req.params.id));

// Root fix: the domain type can only hold the right thing
type User = { id: UserId; name: string };
function findUser(id: UserId): User | undefined { /* ... */ }

const id = parseUserId(req.params.id); // UserId | undefined
```

After the root fix, passing a raw string anywhere is a compile error, not just at the one call site that broke.

## Derive, don't redeclare

```ts
// Don't: a second copy of User's fields that will drift
interface CreateUserInput {
  name: string;
  email: string;
  role: "admin" | "member";
}

// Do: derive from the source of truth
type CreateUserInput = Omit<User, "id" | "createdAt">;
type PublicUser = Pick<User, "id" | "name">;
type CreateUserResult = Awaited<ReturnType<typeof createUser>>;

// Best, with a schema: derive both from it
const CreateUserSchema = UserSchema.omit({ id: true, createdAt: true });
type CreateUserInput = z.infer<typeof CreateUserSchema>;
```

## `readonly` by default

```ts
// Don't: sorts the shared array in place, every caller sees the new order
export function getAllUsers() {
  return users.sort((a, b) => a.name.localeCompare(b.name));
}

// Do: return a sorted copy, typed so callers can't mutate it
export function getAllUsers(): readonly User[] {
  return users.toSorted((a, b) => a.name.localeCompare(b.name));
}
```

`toSorted` needs Node 20+ and `"lib": ["ES2023"]`. Otherwise, use `[...users].sort(...)`.

## No exported mutable state

```ts
// Don't: any module can reassign or mutate the store
export let users: User[] = [];

// Do: the module owns the state, callers get functions
const users: User[] = [];

export function listUsers(): readonly User[] {
  return users;
}

export function insertUser(user: User): void {
  users.push(user);
}
```

## Expected failures are values

```ts
// Don't: "not found" is normal, but it's thrown like a crash
function getUser(id: UserId): User {
  const user = users.find((u) => u.id === id);
  if (!user) throw new Error("not found");
  return user;
}

// Do: absence is part of the return type
function getUser(id: UserId): User | undefined {
  return users.find((u) => u.id === id);
}

// Do: several expected outcomes, as a result union
type DeleteResult = { ok: true } | { ok: false; error: "not_found" };

function deleteUser(id: UserId): DeleteResult {
  const index = users.findIndex((u) => u.id === id);
  if (index === -1) return { ok: false, error: "not_found" };
  users.splice(index, 1);
  return { ok: true };
}
```

The caller must handle every outcome, and the compiler checks it. Throw only for bugs and broken invariants.

## `catch` is `unknown`

```ts
// Don't
} catch (err: any) {
  res.status(500).json({ error: err.message });
}

// Do
} catch (err) {
  const message = err instanceof Error ? err.message : String(err);
  logger.error({ err }, "delete failed");
  res.status(500).json({ error: "Internal error" });
}
```

Also avoid sending raw error messages to clients. Log the detail, return a generic message.

## Never swallow errors

```ts
// Don't: a crash looks like "there are no users"
try {
  res.json(getAllUsers());
} catch (e) {
  console.log(e);
  res.json([]);
}

// Do: let it reach the error middleware, which logs it and returns a 500
res.json(getAllUsers());
```

A `catch` is only allowed when it rethrows, translates to a typed error, or handles one specific known case.

## No floating promises

```ts
// Don't: if this rejects, nothing notices (or the process crashes)
sendWelcomeEmail(user.email);

// Do: await it
await sendWelcomeEmail(user.email);

// Do: intentional fire-and-forget, with its own error handling
void sendWelcomeEmail(user.email).catch((err) =>
  logger.error({ err }, "welcome email failed"),
);
```

## No async where it is ignored

```ts
// Don't: forEach ignores the promises, so nothing waits and errors are lost
users.forEach(async (u) => {
  await notify(u);
});

// Do: sequential, when order matters
for (const u of users) {
  await notify(u);
}

// Do: parallel, when it doesn't
await Promise.all(users.map((u) => notify(u)));
```

The same applies to Express 4 handlers: an `async` handler that rejects is not caught by Express 4. Catch inside the handler, use a wrapper, or upgrade to Express 5, which forwards rejected promises to the error middleware.

## Parallel when independent

```ts
// Don't: three independent calls, one after another
const user = await getUser(id);
const orders = await getOrders(id);
const prefs = await getPrefs(id);

// Do
const [user, orders, prefs] = await Promise.all([getUser(id), getOrders(id), getPrefs(id)]);
```

## Real tests

```ts
// Don't: mocks our own module, so it tests the wiring, not the behavior
vi.spyOn(store, "addUser");
await request(app).post("/users").send(body);
expect(store.addUser).toHaveBeenCalled();

// Do: call the real thing and check what a client would see
const res = await request(app).post("/users").send(body);
expect(res.status).toBe(201);
expect((await request(app).get(`/users/${res.body.id}`)).body.name).toBe("Dan");
```

More in the `tdd` skill's `tests.md` and `mocking.md`.

## No stray logging

```ts
// Don't: unstructured, no context, can't be filtered or turned off
console.log("sending welcome email to " + email);

// Do: the project's logger, structured, with an id to trace
logger.info({ userId: user.id }, "welcome email queued");
```

Don't log secrets or personal data (passwords, tokens, full emails) unless the project's policy allows it.
