# Subtract before you add

The best change is the smallest one that fully solves the problem.

- Before adding code, look for code to delete: dead paths, duplicate helpers, flags nobody sets.
- Reuse what exists before writing something new. Search the codebase first.
- Make the smallest change the evidence justifies. No speculative options, no "while I'm here" refactors.
- When two designs feel tied, pick the one with the smaller API and the cleaner boundary.
- If workarounds keep piling up, stop patching and redesign the piece as if it had always been planned that way.
