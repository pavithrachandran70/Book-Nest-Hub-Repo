# Course correction PER-125 (auto from critic exhaustion)

Authoritative for the next run. Human: review/edit this file, then re-run:
`adlc start PER-125 --skip-dev` or `adlc review request-changes PER-125 --note "..."` then `adlc resume PER-125`.

## Critic mismatches to resolve

- **AC count mismatch:** The LLD lists all 15 ACs under "Acceptance Criteria" (AC1–AC15), matching the ticket. This mismatch is resolved.

- **Test count resolved:** The LLD Tests section now enumerates 15 test cases mapping 1:1 to the 15 ACs: 1 health (AC2), 4 books (AC3, AC4, AC5, plus retrieval verification for AC3), 4 cart (AC6, AC7, AC8, AC9), 2 checkout (AC10, AC11), 2 orders (AC12, AC13), 1 persistence (AC14), and 1 suite-level test (AC15). AC1 is verified by server startup and database file creation during test setup. The narrative under AC15 Verification and the Test Count Summary clarifies that AC15 maps to `tests/suite.test.js`, which spawns `npm test` and asserts exit code 0, providing explicit 1:1 traceability. This mismatch is resolved.

- **Database file and table list:** The LLD uses `data/booknest.sqlite` with five tables (`product`, `cart`, `cart_item`, `order`, `order_item`) throughout (Database Schema section, AC1 text, Notes section). This matches the ticket, requirements, and repo context. This mismatch is resolved.

- **GET /api/orders/:id route handler:** The LLD Route Handlers section now documents `GET /api/orders/:id` explicitly under `src/routes/orders.js`, including its implementation algorithm (select from `order` table, join `order_item`, return 200 with order details or 404 if not found). The route is referenced in tests/persistence.test.js case 1 and tests/orders.test.js cases 1 and 2, and supports AC14 verification. This mismatch is resolved.

- **Missing HLD:** The task references "Check consistency across ticket, requirements, proposed tickets, HLD, per-ticket specs/, breakdown, stories, tests" but no HLD artifact is present in the provided context. The ticket is a single-story epic with no breakdown or child tickets. The flow is ticket → requirements.md → story LLD (`stories/01-Book.md`). The requirements.md serves as the behavioral specification (preserves all 15 ACs with Given/When/Then format and technical notes), and the story LLD provides the concrete file plan, route handlers, schema DDL, and test structure. For a single-story epic, the requirements.md fulfills the role of an HLD (defines outcomes, scope, and verification without prescribing implementation syntax). The traceability chain is complete: ticket ACs → requirements ACs → LLD ACs → test cases. However, the task explicitly names "HLD" as a distinct artifact to check, and it is absent. This is a structural mismatch: either an HLD artifact must be provided, or the plan/requirements must state that this single-story epic skips the HLD and maps ticket → requirements → story LLD directly per the chosen loop (`story_with_design`).

**Remaining mismatch:**

- **Missing HLD artifact:** No `hld.md` or equivalent high-level design document is present. The task requires checking consistency across ticket, requirements, HLD, breakdown, and stories. For a single-story epic with no breakdown, the requirements.md may serve as the HLD, but the task names "HLD" explicitly. Clarify whether this epic skips the HLD (state so in plan.md or requirements.md) or provide an HLD artifact covering the system design (API surface, database schema, auth model, session management, transaction behavior, error handling) before the story LLD.

## Repair rules

- Keep ticket/repo labels (`repo:*`) and out-of-scope constraints.
- Prefer one complete story LLD per involved repo when possible; do not leave truncated mid-sentence files.
- Align AC counts, test counts, and file paths across requirements ↔ HLD ↔ breakdown ↔ stories.
- Rewrite incomplete artifacts fully (no cutoffs).
