# Plan — PER-125 (v1)

- made from: ticket
- source: LLM planner
- needs plan review: yes
- track hint: standard
- loop: story_with_design
- delivery: delivery
- PR fixes run automatically: pr_fix_minor, pr_fix_moderate (other PR comments go to a person)

Build a complete Node.js bookstore API from scratch in a single repo with catalog, cart, checkout, and shipping—a well-defined implementation task that needs a concrete file plan before coding.

## Steps

1. [ ] **analyst (inline_story)** — Transform the ticket's detailed acceptance criteria into a structured Given/When/Then story with technical notes.
   - focus: Keep the AC granular and organized by capability: scaffold, health, catalog CRUD with auth, cart operations, checkout with stock validation, shipping with auth, persistence, and tests. Call out the SQLite schema creation on startup, x-role header auth pattern, and x-cart-id session tracking.
2. [ ] **designer (lld)** — Produce a complete file plan, database schema, route structure, and implementation sequence before any code is written.
   - focus: Design the SQLite schema with product, cart, cart_item, and order tables. Map each AC to specific Express routes with their handlers. Plan the project structure: package.json, src/server.js, src/routes/, src/db/, and tests/. Detail the auth middleware for x-role and cart resolution for x-cart-id. Outline the transaction handling for checkout (stock reduction + order creation).
3. [ ] **test_architect (tests)** — Map each acceptance criterion to a concrete test case covering health, catalog auth, cart flows, checkout validation, stock updates, shipping auth, and persistence.
   - focus: Create test cases for: API health check, admin-only book creation, shopper catalog read, cart item addition with auto-increment, quantity-above-stock rejection, cart item deletion, successful checkout with stock deduction and cart clear, checkout failure when stock is insufficient, admin-only shipping, shopper 403 on shipping, and persistence after restart.
4. [ ] **critic (consistency_check)** — Verify the inline story, LLD, and test plan agree on the schema, routes, auth rules, cart semantics, and checkout transaction behavior.
5. [ ] **architect_gate (review)** _(auto)_ — Human approval of specs and this plan before Dev.
6. [ ] **dev** _(auto)_ — Loop delivery.
7. [ ] **pre_pr_qa** _(auto)_ — Loop delivery.
8. [ ] **open_pr** _(auto)_ — Loop delivery.
9. [ ] **pr_review** _(auto)_ — Loop delivery.
10. [ ] **merge** _(auto)_ — Loop delivery.

## Skipped roles

- **tpm_gate** — The ticket clearly defines scope, acceptance criteria, and what is out of scope; no ambiguity requiring product sign-off.
