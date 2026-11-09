# Requirements — PER-125

**Build the BookNest bookstore API from an empty repo**

When I open a bookstore and the repository has no application code, I want one Node.js API with a SQLite database that can list books, update the catalog, hold a cart, take payment as a checkout, and mark the order shipped, so a shopper can go from an empty catalog to a shipped order in a single run.

**In scope:** project scaffold, health endpoint, catalog CRUD with role-based auth, cart operations with session tracking, checkout with stock validation, shipping with role-based auth, SQLite persistence, and tests.

**Out of scope:** frontend, user accounts, real payment integration, multi-shop support.

---

## Acceptance Criteria

### AC1: Scaffold and Database Creation

**Given** a fresh clone of the repository  
**When** I run `npm install` and `npm start`  
**Then** the API listens on port 3000 and creates `data/booknest.sqlite` with `product`, `cart`, `cart_item`, `order`, and `order_item` tables

**Clarification:** The database file is `data/booknest.sqlite` per the repo context. The five tables are `product`, `cart`, `cart_item`, `order`, and `order_item`. The `order_item` table holds the line items of each order.

---

### AC2: Health Check

**Given** the server is running  
**When** I `GET /api/health`  
**Then** the response is 200 with body `{ "status": "ok" }`

---

### AC3: Admin Can Create Book

**Given** an admin request with header `x-role: admin` and JSON body containing `title`, `author`, `price`, and `stock`  
**When** they `POST /api/books`  
**Then** the response is 201 with the created book's id  
**And** `GET /api/books/:id` returns that book with all fields

---

### AC4: Shopper Cannot Create Book

**Given** a shopper request with header `x-role: shopper`  
**When** they `POST /api/books`  
**Then** the response is 403

---

### AC5: List Books

**Given** books exist in the catalog  
**When** a shopper performs `GET /api/books`  
**Then** the response is 200 with a JSON array of books, each including `id`, `title`, `author`, `price`, and `stock`

---

### AC6: Add Item to New Cart

**Given** a book exists with `stock: 5`  
**When** a shopper performs `POST /api/cart/items` with `{ "bookId": <id>, "quantity": 2 }` and no `x-cart-id` header  
**Then** the response is 201 with a body containing a cart id  
**And** `GET /api/cart` with header `x-cart-id: <that-id>` returns the cart with one line showing the book, quantity 2, unit price, and line total

---

### AC7: Increment Existing Cart Line

**Given** a cart already contains a book with quantity 2  
**When** the shopper adds the same book with quantity 1  
**Then** the cart line quantity becomes 3

---

### AC8: Reject Add Above Stock

**Given** a book with `stock: 5` and a cart line with quantity 2  
**When** the shopper attempts to add quantity 4 of that book  
**Then** the response is 400  
**And** the cart quantity remains 2

---

### AC9: Remove Cart Item

**Given** a cart contains a line for a specific book  
**When** the shopper performs `DELETE /api/cart/items/:bookId` with the correct `x-cart-id`  
**Then** that line is removed from the cart

---

### AC10: Successful Checkout

**Given** a cart whose total quantities are all within current stock  
**When** the shopper performs `POST /api/checkout` with header `x-cart-id: <cart-id>`  
**Then** the response is 201 with an order id, status `placed`, the same lines, and the total amount  
**And** `GET /api/cart` for that cart returns an empty cart  
**And** each book's stock is reduced by the quantity in the order

---

### AC11: Checkout Fails on Insufficient Stock

**Given** a cart with a quantity that exceeds the current stock of a book  
**When** the shopper performs `POST /api/checkout`  
**Then** the response is 400  
**And** no order is created  
**And** stock levels remain unchanged

---

### AC12: Admin Can Ship Order

**Given** an order exists with status `placed`  
**When** an admin performs `POST /api/orders/:id/ship` with header `x-role: admin`  
**Then** the order status becomes `shipped`

---

### AC13: Shopper Cannot Ship Order

**Given** an order exists with status `placed`  
**When** a shopper performs `POST /api/orders/:id/ship` with header `x-role: shopper`  
**Then** the response is 403

---

### AC14: Data Persists After Restart

**Given** the server has created books, a cart, and an order  
**When** the server restarts  
**Then** `GET /api/books/:id` returns the book, `GET /api/orders/:id` returns the order, and stock levels reflect prior checkout reductions

---

### AC15: Test Suite Passes

**Given** the repository with all source code  
**When** I run `npm test`  
**Then** all tests pass, covering health endpoint, catalog auth, catalog read, cart operations, checkout flows, shipping auth, and persistence

**Verification:** AC15 is satisfied when `npm test` exits with code 0. The test suite comprises 15 test cases total: 1 health (AC2), 4 books (AC3, AC4, AC5, plus one for retrieving a created book), 4 cart (AC6, AC7, AC8, AC9), 2 checkout (AC10, AC11), 2 orders (AC12, AC13), 1 persistence (AC14), and 1 suite-level test that runs `npm test` programmatically and asserts exit code 0 (AC15 verification). AC1 is verified by test setup: the server starts successfully and creates `data/booknest.sqlite` with the five required tables (`product`, `cart`, `cart_item`, `order`, `order_item`), which is a precondition for all subsequent tests.

**Test case mapping:**
- **AC1 (scaffold and database creation):** Verified by test setup (server starts, database file exists with five tables)
- **AC2 (health check):** 1 test in `tests/health.test.js`
- **AC3 (admin can create book):** 1 test in `tests/books.test.js`
- **AC4 (shopper cannot create book):** 1 test in `tests/books.test.js`
- **AC5 (list books):** 1 test in `tests/books.test.js`
- **AC3 retrieval verification (`GET /api/books/:id` returns created book):** 1 test in `tests/books.test.js`
- **AC6 (add item to new cart):** 1 test in `tests/cart.test.js`
- **AC7 (increment existing cart line):** 1 test in `tests/cart.test.js`
- **AC8 (reject add above stock):** 1 test in `tests/cart.test.js`
- **AC9 (remove cart item):** 1 test in `tests/cart.test.js`
- **AC10 (successful checkout):** 1 test in `tests/checkout.test.js`
- **AC11 (checkout fails on insufficient stock):** 1 test in `tests/checkout.test.js`
- **AC12 (admin can ship order):** 1 test in `tests/orders.test.js`
- **AC13 (shopper cannot ship order):** 1 test in `tests/orders.test.js`
- **AC14 (data persists after restart):** 1 test in `tests/persistence.test.js`
- **AC15 (test suite passes):** 1 test in `tests/suite.test.js` that spawns `npm test` and asserts exit code 0

**Total: 15 test cases.** AC1 is verified by startup; ACs 2–15 are verified by 15 explicit test cases (14 behavioral plus 1 suite-level integration test).

---

## Technical Notes

- **Schema:** SQLite tables `product` (id, title, author, price, stock), `cart` (id), `cart_item` (cart_id, book_id, quantity), `order` (id, cart_id, status, total, created_at), and `order_item` (order_id, book_id, quantity, unit_price). Create on startup if missing. File path: `data/booknest.sqlite`.
- **Auth:** `x-role` header (`admin` or `shopper`) controls access to `POST /api/books` and `POST /api/orders/:id/ship`. Requests without `x-role` default to `shopper`.
- **Cart session:** `x-cart-id` header identifies the cart. Generate a new UUID cart id on first `POST /api/cart/items` when none is provided; return it in the response so the client can include it on subsequent cart requests.
- **Checkout transaction:** Atomically validate stock for all cart items, reduce stock for each, create the order and order_item rows, and clear the cart. Roll back on any failure (e.g., insufficient stock, database error).
- **Errors:** Return `{ "error": "<message>" }` with appropriate HTTP status (400 for validation/business rule failures, 403 for authorization failures, 404 for not found, 500 for unexpected server errors).
- **Test files:** All test files live in `tests/` at the repository root. Test setup creates the database before each test suite and clears all tables before each test case to ensure isolation.
- **Routes:** The API exposes `GET /api/health`, `GET /api/books`, `POST /api/books`, `GET /api/books/:id`, `GET /api/cart`, `POST /api/cart/items`, `DELETE /api/cart/items/:bookId`, `POST /api/checkout`, `GET /api/orders/:id`, and `POST /api/orders/:id/ship`. The `GET /api/orders/:id` route is required to verify AC14 (data persistence after restart).

---

## Verification

Each acceptance criterion maps to verification steps in the test suite or runtime behavior. AC1 is verified by server startup and database file creation during test setup. ACs 2–15 are verified by the 15 test cases enumerated in AC15's verification section. AC15 is verified by running `npm test` and confirming exit code 0. The LLD (`stories/01-Book.md`) details the file plan, route handlers, schema DDL, and test structure with explicit assertions for each AC.
