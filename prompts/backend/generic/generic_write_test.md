---
name: Test
dcc_uri: dev/prompts/generic_write_test
description: Generate tests for backend logic, services, or API handlers
invokable: true
version: '0.1'
dcc_tags:
  - dev
  - test
---

Generate tests for the selected backend code.

Cover in this order:
1. **Happy path** — valid input, expected output/side effect
2. **Validation failures** — missing fields, bad types, out-of-range values
3. **Not found / empty** — resource doesn't exist, empty result set
4. **External failure** — downstream service/DB throws or times out
5. **Idempotency** (if applicable) — calling twice produces the same result

Rules:
- Unit tests for pure logic and service methods — mock external dependencies
- Integration tests for persistence and protocol boundaries — use real DB/containers, not in-memory fakes that differ from production
- Each test has a single clear assertion focus
- Name tests: `should [expected outcome] when [condition]`
- Do not test framework behaviour — test your code's response to it
- Every bug fix warrants a regression test that would have caught it
