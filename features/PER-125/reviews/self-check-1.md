# FAIL

## Concrete Mismatches

1. **AC count mismatch:** The story LLD lists 15 ACs under "Acceptance Criteria" (AC1–AC15), matching the ticket's 15 bullet ACs. **Resolved** — the LLD preserves all 15 ACs verbatim from the ticket.

2. **Test count narrative inconsistency:** The LLD's Tests section states "AC15 ('run `npm test` and all tests pass') is verified by the test runner's exit code: when all 14 behavioral tests pass, `npm test` exits with code 0, fulfilling AC15. No 15th dedicated test case is required." However, the ticket's AC15 is a concrete acceptance criterion: "Given the repo, when I run `npm test`, then tests covering health, catalog auth, cart, checkout, stock, and ship all pass." The LLD's narrative treats AC15 as a meta-statement about the other 14 tests, rather than as a standalone AC. **Resolution required:** Either (a) add a 15th test case that runs `npm test` programmatically and asserts exit code 0, or (b) clarify in the Tests section that AC15 is verified by the successful execution of the suite (exit code 0) without demoting AC15 from the AC list or calling it a "meta-statement." The current phrasing "No 15th dedicated test case is required" contradicts the ticket's explicit AC15 requirement.

3. **Database file and table list:** The LLD correctly specifies `data/booknest.sqlite` with five tables (`product`, `cart`, `cart_item`, `order`, `order_item`), matching the ticket's corrected AC1 and the repo context. **Resolved** — no mismatch.

4. **GET /api/orders/:id route handler:** The LLD's Route Handlers section explicitly documents `router.get('/api/orders/:id', (req, res) => { /* get order by id */ })` with full specification (query `order` table, join `order_item`, return 200 with order details or 404 if not found). **Resolved** — the route is documented.

5. **Missing HLD artifact:** The task requires checking consistency across "ticket, requirements, proposed tickets, HLD, per-ticket specs/, breakdown, stories, tests, and merged_plan.md when present." No HLD artifact is present in the context. Per the task instructions, the HLD should describe outcomes in abstract terms (e.g., "response format `{ "status": "ok" }`"), not language-specific syntax. The story LLD is present and detailed, but it is a low-level design with concrete file paths and code snippets, not a high-level design describing outcomes. **Resolution required:** Either (a) provide the missing HLD artifact describing the API's behavior in outcome terms (e.g., "Health check returns status object with 'ok' value," "Admin creates book, shopper reads it," etc.), or (b) clarify in the plan that this single-story epic maps ticket → requirements → story LLD directly, skipping the HLD step, and update the task's consistency check requirement to reflect that deviation.

6. **change_requests.md compliance:** The change_requests.md file lists five mismatches to resolve. The LLD addresses four of them (AC count, database file/tables, GET /api/orders/:id route, test count narrative attempt). However, the test count narrative resolution is incomplete (see item 2 above), and the missing HLD (item 5 above) was not addressed. **Resolution required:** Fully resolve the test count narrative per item 2, and either provide the HLD or clarify the plan's deviation from the expected artifact set.

---

**Summary:** The LLD has resolved the AC count, database file/tables, and GET /api/orders/:id route documentation. Two mismatches remain:

- **Test count narrative (AC15 verification):** The LLD's explanation treats AC15 as a delivery gate rather than a concrete AC with a test case or clear verification method. The ticket's AC15 requires the test suite to exist and pass; the LLD must either add a 15th test case or state that AC15 is verified by the suite's exit code without demoting it from the AC list.
- **Missing HLD:** The task's consistency check expects an HLD artifact. Either provide the HLD or update the plan to clarify that this epic skips the HLD and maps ticket → requirements → story LLD directly.
