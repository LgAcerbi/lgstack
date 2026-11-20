# Foundational thinking

Logic is easy to change. Data shapes are not. Settle the shapes first.

- Each piece of state has exactly one owner. If two places can write it, the design is not done.
- Each task has one way to do it. Two code paths for the same job will drift apart.
- Callers should need to know as little as possible. Push complexity behind a small interface.
- Put rules in data structures where you can. A type that cannot hold a bad value beats a check that rejects one.
- Compare at least two real shapes before picking one. The first idea is rarely the best one.
