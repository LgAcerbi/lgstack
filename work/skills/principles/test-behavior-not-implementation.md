# Test behavior, not implementation

A test should survive any refactor that keeps behavior the same.

- Test through public interfaces. Never test private functions or internal call order.
- Expected values come from an independent source: a literal, a worked example, the spec. Never recompute them the way the code does.
- A test name says what the caller gets ("user can check out with a valid cart"), not how the code does it.
- Details and examples: the `tdd` skill.
