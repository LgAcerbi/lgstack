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
