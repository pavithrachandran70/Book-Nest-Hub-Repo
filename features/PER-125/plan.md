# Plan — PER-125 (v2)

- made from: ticket
- source: LLM planner
- needs plan review: yes
- track hint: quick
- loop: story_with_design
- delivery: delivery
- PR fixes run automatically: pr_fix_minor, pr_fix_moderate (other PR comments go to a person)

Build the complete BookNest API from scratch in one story: this is a single-repo, self-contained implementation with clear acceptance criteria and explicit file structure, fitting quick_fix with designer for the concrete file plan.

## Steps

1. [x] **analyst (inline_story)** — Convert the ticket's 15 acceptance criteria into one implementable story with Given/When/Then format and tech notes.
   - focus: Preserve all 15 ACs verbatim from the ticket. AC15 ('run npm test and all tests pass') is a concrete acceptance criterion, not a meta-gate. Use the corrected database file name 'data/booknest.sqlite' and the five tables: product, cart, cart_item, order, order_item.
2. [x] **designer (lld)** — Provide a concrete file plan, route handlers, schema DDL, and test structure before coding.
   - focus: List all 15 ACs in the story's Acceptance Criteria section. Document the GET /api/orders/:id route handler explicitly. Map AC15 to a test suite integration check or clarify it as verified by the test runner's exit code. Reconcile the test count narrative: if AC15 is verified by the 14 behavioral tests' collective pass, state that clearly without demoting AC15 from the AC list.
3. [x] **test_architect (tests)** — Map each of the 15 acceptance criteria to test cases with 1:1 traceability.
   - focus: Ensure AC15 has a corresponding test entry (either a dedicated suite runner test or a narrative stating AC15 is verified by the suite's exit code). Resolve the test count mismatch: ticket AC15 requires tests to pass; the Tests section must reflect this without ambiguity.
4. [ ] **critic (consistency_check)** — Cross-check the story LLD against the ticket and change_requests.md to ensure AC count, test count, file paths, and database details are consistent.
   - focus: Verify all 15 ACs appear in the LLD, the GET /api/orders/:id route is documented, the database file is 'data/booknest.sqlite' with five tables, and the test count narrative is resolved. Address every mismatch listed in change_requests.md.
5. [ ] **architect_gate (review)** _(auto)_ — Human approval of specs and this plan before Dev.
6. [ ] **dev** _(auto)_ — Loop delivery.
7. [ ] **pre_pr_qa** _(auto)_ — Loop delivery.
8. [ ] **open_pr** _(auto)_ — Loop delivery.
9. [ ] **pr_review** _(auto)_ — Loop delivery.
10. [ ] **merge** _(auto)_ — Loop delivery.

## Skipped roles

- **tpm_gate** — Ticket scope and acceptance criteria are explicit and technical; no product ambiguity or cross-team coordination needed.
