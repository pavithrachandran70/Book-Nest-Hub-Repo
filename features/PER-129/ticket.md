---
key: PER-129
type: story
status: todo
blocks: []
blocked_by: []
labels: []
repos: []
summary: Build the BookNest bookstore API from an empty repo
---

When I open BookNest and the repository has no application code, I want one Node.js API with a SQLite database that can list books, update the catalog, hold a cart, take payment as a checkout, and mark the order shipped, so a shopper can go from an empty catalog to a shipped order in a single run.

In scope: project scaffold, SQLite schema, catalog, cart, checkout, shipping, and tests.

Out of scope: frontend, accounts, real payments, more than one shop.

Acceptance criteria:

* Given a fresh clone, when I run npm install and npm start, then the API listens on port 3000 and creates data/booknest.sqlite with product, cart, cart_item, order, and order_item tables.
* Given the server is running, when I GET /api/health, then the response is 200 and { "status": "ok" }.
* Given an admin request with x-role: admin and a title, author, price, and stock, when they POST /api/books, then the response is 201 and GET /api/books/:id returns that book.
* Given a shopper request with x-role: shopper, when they POST /api/books, then the response is 403.
* Given books exist, when a shopper GET /api/books, then the response is 200 and the list includes title, author, price, and stock.
* Given a book with stock 5, when a shopper POST /api/cart/items with quantity 2 and no x-cart-id, then the response is 201, includes a cart id, and GET /api/cart with that x-cart-id shows the line, the price, and the line total.
* Given that cart already contains the book, when the shopper adds quantity 1 again, then the line quantity becomes 3.
* Given a quantity above stock, when the shopper adds it, then the response is 400 and the cart is unchanged.
* Given a cart line, when the shopper DELETE /api/cart/items/:bookId, then that line is removed.
* Given a cart whose quantities are still in stock, when the shopper POST /api/checkout with that x-cart-id, then the response is 201 with an order id, status placed, the same lines, and the order total.
* Given checkout succeeded, when I GET /api/cart for that cart, then the cart is empty, and each book's stock is reduced by the ordered quantity.
* Given a cart quantity above current stock, when I checkout, then the response is 400 and no order is written.
* Given an order is placed, when an admin POST /api/orders/:id/ship, then the status becomes shipped. A shopper calling that route gets 403.
* Given the server restarts, when I read the book, the order, and the reduced stock, then the SQLite data is still there.
* Given the repo, when I run npm test, then tests covering health, catalog auth, cart, checkout, stock, and ship all pass.
