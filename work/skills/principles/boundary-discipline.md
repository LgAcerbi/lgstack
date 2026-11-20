# Boundary discipline

Validate once, where data crosses into the system. Trust types everywhere else.

- Boundaries are HTTP requests and responses, `JSON.parse`, files, environment variables, database rows, `postMessage`, user input and third-party SDKs.
- At the boundary, parse into a named domain type with the project's schema library (zod, valibot or similar). Data is `unknown` until parsed.
- Converting is not validating. `Number()`, `parseInt`, `String()` and `Boolean()` coerce bad input into a plausible value (`Number("0x1")` is `1`, `Number("")` is `0`). Parse strictly against the exact accepted format and reject everything else. Path params, query strings and headers are boundaries too.
- A value should have one accepted form. If several inputs reach the same record (`/users/1`, `/users/1.0`, `/users/0x1`), the parse is too loose.
- Inside the boundary, do not re-check what the types already guarantee. Defensive checks on trusted paths hide real bugs and add noise.
- Mock only at boundaries in tests. Never mock your own modules.
