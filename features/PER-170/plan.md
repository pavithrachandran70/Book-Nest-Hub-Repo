# Plan — PER-170 (v1)

- made from: ticket
- source: LLM planner
- needs plan review: yes
- track hint: quick
- loop: quick_fix
- delivery: delivery
- PR fixes run automatically: pr_fix_minor, pr_fix_moderate (other PR comments go to a person)

Fix exponent() for Rational numbers to avoid float conversion and return correct values for rationals just below 1; quick_fix fits because this is a single-repo behavioral fix with clear acceptance criteria in the ticket.

## Steps

1. [ ] **analyst (inline_story)** — Turn the ticket into a single implementable story with Given/When/Then acceptance criteria.
   - focus: Focus on the edge case where Rational is just below 1, ensure the exponent method bypasses float conversion, and map each acceptance criterion to a concrete test case.
2. [ ] **architect_gate (review)** _(auto)_ — Human approval of specs and this plan before Dev.
3. [ ] **dev** _(auto)_ — Loop delivery.
4. [ ] **pre_pr_qa** _(auto)_ — Loop delivery.
5. [ ] **open_pr** _(auto)_ — Loop delivery.
6. [ ] **pr_review** _(auto)_ — Loop delivery.
7. [ ] **merge** _(auto)_ — Loop delivery.

## Skipped roles

- **critic** — Only one spec document (the inline story) is produced, so there are no multiple artifacts to cross-check for consistency.
