# Plan — PER-233 (v1)

- made from: ticket
- source: LLM planner
- needs plan review: yes
- track hint: standard
- loop: story_with_design
- delivery: delivery
- PR fixes run automatically: pr_fix_minor, pr_fix_moderate (other PR comments go to a person)

Add a single GET endpoint to an existing Express API; one inline story with a file plan is sufficient.

## Steps

1. [x] **analyst (inline_story)** — Convert the ticket into a single implementable story with Given/When/Then acceptance criteria.
   - focus: Focus on the exact response shapes for 200 and 404, and the requirement that both shopper and admin roles can call it.
2. [x] **designer (lld)** — Provide a file plan showing which route file to modify, the SQL query needed, and the response mapping.
   - focus: Show where the route goes in the Express app structure, the exact SELECT query to fetch order and items, and how to check x-role.
3. [x] **test_architect (tests)** — Edge cases include unknown order ID, role authorization for both shopper and admin, and correct item structure.
   - focus: Cover 200 with all fields present, 404 for missing order, and both shopper and admin calling successfully.
4. [ ] **architect_gate (review)** _(auto)_ — Human approval of specs and this plan before Dev.
5. [ ] **dev** _(auto)_ — Loop delivery.
6. [ ] **pre_pr_qa** _(auto)_ — Loop delivery.
7. [ ] **open_pr** _(auto)_ — Loop delivery.
8. [ ] **pr_review** _(auto)_ — Loop delivery.
9. [ ] **merge** _(auto)_ — Loop delivery.

## Skipped roles

- **tpm_gate** — Scope and acceptance criteria are fully specified in the ticket; no product ambiguity.
- **critic** — Only one spec document (the story with design and tests); nothing to cross-check.
