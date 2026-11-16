# Plan — PER-267 (v1)

- made from: ticket
- source: LLM planner
- needs plan review: yes
- track hint: standard
- loop: feature_with_architecture
- delivery: delivery
- PR fixes run automatically: pr_fix_minor, pr_fix_moderate (other PR comments go to a person)

Build the first version of the BookNest app with book search capability; this is a new application in an empty repo, so architecture design is needed.

## Steps

1. [ ] **analyst (requirements)** — Extract precise requirements including the Book shape, search behavior, and GraphQL gateway constraint.
   - focus: Focus on the search endpoint contract, the Book data model, and the constraint that the web app calls only the GraphQL gateway.
2. [ ] **architect (hld)** — Design the first version of the application: schema, REST API contract, and component structure.
   - focus: Define the SQLite schema for books and the demo user's cart, the REST endpoints including search, and how the app initializes the database on startup. Address the GraphQL gateway constraint mentioned in AC5.
3. [ ] **scrum_master (breakdown)** — Break down into implementable stories: schema setup, book endpoints, search logic, and tests.
   - focus: Split into stories for database setup, core book CRUD, search with case-insensitive filtering, and cart integration for the 'in cart' indicator.
4. [ ] **designer (lld)** — Provide file-by-file implementation plans for each story in a new codebase.
   - focus: Detail the Express route structure, SQLite query patterns, and test organization for the new application.
5. [ ] **test_architect (tests)** — Map acceptance criteria to concrete test cases covering search behavior, edge cases, and the Book shape.
   - focus: Ensure tests cover case-insensitive matching, empty search, no results, and the full Book shape in responses.
6. [ ] **merge_pack (pack)** _(auto)_ — Loop feature_with_architecture: Deterministic merged_plan.md from ordered specs and story summaries.
7. [ ] **critic (consistency_check)** — Cross-check that requirements, HLD, breakdown, and stories agree on the Book schema, search contract, and GraphQL constraint.
8. [ ] **architect_gate (review)** _(auto)_ — Human approval of specs and this plan before Dev.
9. [ ] **dev** _(auto)_ — Loop delivery.
10. [ ] **pre_pr_qa** _(auto)_ — Loop delivery.
11. [ ] **open_pr** _(auto)_ — Loop delivery.
12. [ ] **pr_review** _(auto)_ — Loop delivery.
13. [ ] **merge** _(auto)_ — Loop delivery.

## Skipped roles

- **tpm_gate** — The ticket clearly states scope and acceptance criteria; no ambiguity or cross-team coordination needed.

## Planner notes

- attempt 1: loop feature_with_architecture is ruled out: you answered that its not-when [N1] applies (one story's worth of work inside one component, and every interface stays as it is → quick_fix or story_with_design). Pick the loop that line points to, or the next loop down whose not-when lines do not apply
