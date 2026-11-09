# Plan — PER-129 (v1)

- made from: ticket
- source: LLM planner
- needs plan review: yes
- track hint: standard
- loop: feature_with_architecture
- delivery: delivery
- PR fixes run automatically: pr_fix_minor, pr_fix_moderate (other PR comments go to a person)

Building a complete Node.js bookstore API with SQLite from an empty repo requires a data contract (schema), API contract (routes), and comprehensive implementation—feature_with_architecture loop fits because this creates new API and data contracts in a repo with no existing application code.

## Steps

1. [x] **analyst (requirements)** — Extract structured requirements from the detailed acceptance criteria.
   - focus: Keep the requirements focused on the API contract, data model, and behavioral rules; the ticket already provides clear acceptance criteria.
2. [x] **architect (hld)** — Define the SQLite schema, Express route contracts, auth model, and cart/checkout state transitions before coding.
   - focus: Design the complete data model (product, cart, cart_item, order, order_item tables with constraints), API surface (all routes with request/response shapes), role-based access rules, and cart-to-order state machine; address concurrency for stock updates.
3. [x] **scrum_master (breakdown)** — Break down into implementable slices: scaffold, schema, each API endpoint group, and tests.
   - focus: Structure stories so the first delivers the project scaffold and schema, then catalog endpoints, then cart, then checkout and shipping, then comprehensive tests; each story should be independently testable.
4. [x] **designer (lld)** — Each story needs a concrete file plan and implementation detail for the endpoints, database access, and validation logic.
   - focus: Provide file structures, Express route handler skeletons, SQL queries for each operation, and validation rules; ensure better-sqlite3 usage patterns and transaction boundaries are explicit.
5. [x] **test_architect (tests)** — Map each acceptance criterion to specific test cases covering happy paths, auth, validation, and edge cases.
   - focus: Create tests for health check, catalog auth (admin/shopper), cart operations (add/update/delete), checkout with stock validation, shipping auth, and persistence across restarts; ensure traceability to every acceptance criterion.
6. [x] **critic (consistency_check)** — Verify consistency between the schema in HLD, the API contracts in HLD, the route implementations in stories, and the test coverage.
   - focus: Check that every table in the schema appears in stories, every route in the HLD has implementation detail in a story, and every acceptance criterion has a corresponding test case.
7. [x] **merge_pack (pack)** _(auto)_ — Pack per-ticket specs into merged_plan.md for review.
8. [x] **architect_gate (review)** _(auto)_ — Human approval of specs and this plan before Dev.
9. [x] **dev** _(auto)_ — Loop delivery.
10. [x] **pre_pr_qa** _(auto)_ — Loop delivery.
11. [x] **open_pr** _(auto)_ — Loop delivery.
12. [ ] **pr_review** _(auto)_ — Loop delivery.
13. [ ] **merge** _(auto)_ — Loop delivery.

## Skipped roles

- **tpm_gate** — The ticket clearly defines scope, acceptance criteria, and what is out of scope; no product ambiguity exists.
