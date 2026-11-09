# Course correction PER-125 (auto from critic exhaustion)

Authoritative for the next run. Human: review/edit this file, then re-run:
`adlc start PER-125 --skip-dev` or `adlc review request-changes PER-125 --note "..."` then `adlc resume PER-125`.

## Critic mismatches to resolve

- **AC count mismatch:** Ticket lists 15 bullet ACs. Requirements.md lists 15 numbered ACs (AC1–AC15). The LLD lists 14 ACs under "Acceptance Criteria" and states "AC15 is a delivery gate fulfilled by the presence of the 14 behavioral tests below; it is not a separate test case." The ticket's AC15 is a concrete acceptance criterion ("Given the repo, when I run `npm test`, then tests covering health, catalog auth, cart, checkout, stock, and ship all pass") that must be preserved as AC15 in the LLD. The LLD must list all 15 ACs verbatim from the ticket and reconcile the test count narrative separately.

- **Test count mismatch:** Ticket AC15 says "tests covering health, catalog auth, cart, checkout, stock, and ship all pass." Requirements.md AC15 enumerates 14 behavioral tests verifying ACs 1–14 and states "AC15 is a delivery gate requiring those 14 tests to exist and pass; it is not itself a separate test case." The LLD's Tests section lists 14 behavioral tests and repeats "Ticket AC15 ('run `npm test` and all tests pass') is a delivery gate fulfilled by these 14 tests, not a separate test case" in three places. This is inconsistent: the ticket's AC15 is an acceptance criterion requiring the test suite to pass, not a meta-statement about the other ACs. The LLD must preserve AC15 as a standalone AC and reconcile the test count: either AC15 maps to a single integration test that runs `npm test` and asserts all tests pass, or the narrative clarifies that AC15 is verified by the suite's exit code rather than a dedicated test case, without demoting AC15 from the AC list.

- **Resolved: database file and table list.** The ticket now specifies `data/booknest.sqlite` with the five tables `product`, `cart`, `cart_item`, `order`, and `order_item`. This matches the repo context in `context/repos/Book.md`. The requirements and LLD must use this file name and these five tables, and must not restate the conflict.

- **Missing file in file plan:** The LLD File Plan lists `src/routes/orders.js` under Source Files but does not list a `GET /api/orders/:id` route handler in the Route Handlers section. Requirements.md AC14 says "`GET /api/orders/:id` returns the order" (implied by "the order is still readable"). The LLD Tests section, `tests/persistence.test.js` case 1, says "`GET /api/orders/1` returns the order." The LLD must document the `GET /api/orders/:id` route handler in the Route Handlers section, or the requirements and tests must be revised to remove the `GET /api/orders/:id` dependency from AC14's verification.

- **HLD outcome missing:** The ticket describes outcomes ("the response is 201 and `GET /api/books/:id` returns that book," "the response is 200 and the list includes title, author, price, and stock," etc.). The requirements.md preserves these outcomes in each AC. The LLD does not have an HLD section; it is a story LLD. Per the task ("Check consistency across ticket, requirements, proposed tickets, HLD, per-ticket specs/, breakdown, stories, tests"), an HLD artifact is expected. The missing HLD violates the traceability requirement. Either provide the HLD, or clarify that this single-story epic skips the HLD and maps ticket → requirements → story LLD directly.

## Repair rules

- Keep ticket/repo labels (`repo:*`) and out-of-scope constraints.
- Prefer one complete story LLD per involved repo when possible; do not leave truncated mid-sentence files.
- Align AC counts, test counts, and file paths across requirements ↔ HLD ↔ breakdown ↔ stories.
- Rewrite incomplete artifacts fully (no cutoffs).
