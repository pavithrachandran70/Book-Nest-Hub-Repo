# Plan — PER-127 (v1)

- made from: ticket
- source: LLM planner
- needs plan review: yes
- track hint: standard
- loop: story_with_design
- delivery: delivery
- PR fixes run automatically: pr_fix_minor, pr_fix_moderate (other PR comments go to a person)

Full-stack Node.js bookstore API built from scratch in a single repo with catalog, cart, checkout, and shipping; story_with_design fits because the ticket is concrete and self-contained but needs a file plan before coding.

## Steps

1. [ ] **analyst (inline_story)** — Turn the detailed acceptance criteria into a structured Given/When/Then story with technical notes.
   - focus: The ticket already lists concrete acceptance criteria; organize them into testable scenarios and add implementation notes for the Express routes, SQLite schema, and role-based access control.
2. [ ] **designer (lld)** — This greenfield API needs a concrete file plan: package.json, schema DDL, server.js, route modules, middleware, and test structure.
   - focus: Provide the SQLite schema DDL, Express route structure, middleware for role and cart-id headers, and the file layout for src/ and tests/. Keep it simple: plain JavaScript, CommonJS, no TypeScript or Prisma.
3. [ ] **test_architect (tests)** — The ticket explicitly requires tests covering health, catalog auth, cart, checkout, stock, and ship; map each acceptance criterion to a test case.
   - focus: Map every AC to one Jest test case in the Tests section, covering health endpoint, admin vs shopper auth, cart operations, checkout with stock validation, shipping, and persistence after restart.
4. [ ] **architect_gate (review)** _(auto)_ — Human approval of specs and this plan before Dev.
5. [ ] **dev** _(auto)_ — Loop delivery.
6. [ ] **pre_pr_qa** _(auto)_ — Loop delivery.
7. [ ] **open_pr** _(auto)_ — Loop delivery.
8. [ ] **pr_review** _(auto)_ — Loop delivery.
9. [ ] **merge** _(auto)_ — Loop delivery.

## Skipped roles

- **critic** — Single inline story and LLD; no multiple artifacts to cross-check for consistency.
