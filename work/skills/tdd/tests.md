# Good and bad tests

## Good tests

**Integration-style**: test through real interfaces, not mocks of internal parts.

```typescript
// GOOD: tests observable behavior
test("user can checkout with valid cart", async () => {
  const cart = createCart();
  cart.add(product);
  const result = await checkout(cart, paymentMethod);
  expect(result.status).toBe("confirmed");
});
```

Characteristics:

- Tests behavior that users and callers care about
- Uses the public API only
- Survives internal refactors
- Describes what, not how
- One logical assertion per test

## Bad tests

**Implementation-detail tests** are coupled to internal structure.

```typescript
// BAD: tests implementation details
test("checkout calls paymentService.process", async () => {
  const mockPayment = vi.mocked(paymentService);
  await checkout(cart, payment);
  expect(mockPayment.process).toHaveBeenCalledWith(cart.total);
});
```

Red flags:

- Mocking internal collaborators
- Testing private functions
- Asserting on call counts or call order
- Breaking on a refactor that did not change behavior
- A test name that describes how, not what
- Verifying through a side channel instead of the interface

```typescript
// BAD: bypasses the interface to verify
test("createUser saves to database", async () => {
  await createUser({ name: "Alice" });
  const row = await db.query("SELECT * FROM users WHERE name = ?", ["Alice"]);
  expect(row).toBeDefined();
});

// GOOD: verifies through the interface
test("createUser makes user retrievable", async () => {
  const user = await createUser({ name: "Alice" });
  const retrieved = await getUser(user.id);
  expect(retrieved.name).toBe("Alice");
});
```

**Order-dependent tests**: rely on shared state that other tests can change.

```typescript
// BAD: assumes the seed data is untouched; breaks once a DELETE test runs first
test("GET /users/:id returns a user", async () => {
  const res = await request(app).get("/users/2");
  expect(res.body.name).toBe("Alice");
});

// GOOD: each test sets up the state it needs
beforeEach(() => {
  resetUsers([{ id: 2, name: "Alice", email: "alice@example.com", role: "member" }]);
});

test("GET /users/:id returns a user", async () => {
  const res = await request(app).get("/users/2");
  expect(res.body.name).toBe("Alice");
});
```

**Tautological tests**: the expected value restates the implementation, so the test passes by construction.

```typescript
// BAD: expected value is computed the way the code computes it
test("calculateTotal sums line items", () => {
  const items = [{ price: 10 }, { price: 5 }];
  const expected = items.reduce((sum, i) => sum + i.price, 0);
  expect(calculateTotal(items)).toBe(expected);
});

// GOOD: expected value is an independent, known literal
test("calculateTotal sums line items", () => {
  expect(calculateTotal([{ price: 10 }, { price: 5 }])).toBe(15);
});
```
