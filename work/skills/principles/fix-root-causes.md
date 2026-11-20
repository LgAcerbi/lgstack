# Fix root causes

Find why it broke, then fix that.

- A null check, a retry, a `try/catch` or a special case that makes the symptom disappear is not a fix unless it is the cause.
- Trace the bad value back to where it was created. Fix it there.
- If the root cause is out of scope, make the smallest in-scope fix, say so, and name the real cause as open work.
