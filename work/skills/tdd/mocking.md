# When to mock

Mock at **system boundaries** only:

- External APIs (payments, email and so on)
- Databases, sometimes. Prefer a real test database.
- Time and randomness
- The file system, sometimes

Do not mock:

- Your own modules or classes
- Internal collaborators
- Anything you control

## Designing for mockability

At system boundaries, design interfaces that are easy to replace in tests.

**1. Inject dependencies**

Pass external dependencies in instead of creating them inside:

```typescript
// Easy to mock
function processPayment(order: Order, paymentClient: PaymentClient) {
  return paymentClient.charge(order.total);
}

// Hard to mock
function processPayment(order: Order) {
  const client = new StripeClient(process.env.STRIPE_KEY);
  return client.charge(order.total);
}
```

**2. Prefer SDK-style interfaces over a generic fetcher**

Write one function per external operation instead of one generic function with conditional logic:

```typescript
// GOOD: each function is independently mockable and typed
const api = {
  getUser: (id: UserId) => fetch(`/users/${id}`),
  getOrders: (userId: UserId) => fetch(`/users/${userId}/orders`),
  createOrder: (data: NewOrder) => fetch("/orders", { method: "POST", body: JSON.stringify(data) }),
};

// BAD: the mock needs conditional logic inside it
const api = {
  fetch: (endpoint: string, options?: RequestInit) => fetch(endpoint, options),
};
```

With the SDK style:

- Each mock returns one specific shape
- Test setup has no conditional logic
- It is easy to see which endpoints a test exercises
- Each endpoint is type-safe
