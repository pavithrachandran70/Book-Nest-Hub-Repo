---
key: PER-233
type: story
status: todo
blocks: []
blocked_by: []
labels: []
repos: []
summary: "Add\_ `GET /api/orders/:id` \_so a placed order can be read back"
---

Description

Checkout already creates an order and `POST /api/orders/:id/ship` marks it shipped. There is no way to read that order.

Add `GET /api/orders/:id` on the existing BookNest API (port 3001).

* A real order returns 200 and `{ "id", "status", "total", "createdAt", "items" }`. Each item is `{ "bookId", "quantity", "price" }`. `status` is `placed` or `shipped`.
* An unknown id returns 404 and `{ "error": "Order not found" }`.
* A shopper and an admin can both call it. No new tables. Do not change health, books, cart, checkout, or ship.
