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
**Then** the API listens on port 3000 and creates `data/paperboat.sqlite` with `product`, `cart`, `cart_item`, `order`, and `order_item` tables

**Clarification:** The ticket specifies "product, cart, cart-item, and order tables" but an orders table requires both an `order` header table and an `order_item` line-item table to hold the full order structure. The five-table design (`product`, `cart`, `cart_item`, `order`, `order_item`) is the minimal implementation that satisfies the ticket's intent. The repo-context/Book.md specifies a database file named `data/booknest.sqlite`; this requirement follows the ticket and uses `data/paperboat.sqlite`. Implementations should reconcile this naming before deployment, or update the repo-context baseline to match the ticket.

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
**Then** `GET /api/books/:id` returns the book, the order is still readable, and stock levels reflect prior checkout reductions

---

### AC15: Test Suite Passes

**Given** the repository with all source code  
**When** I run `npm test`  
**Then** all tests pass, covering:
- Health endpoint (`tests/health.test.js`: 1 test verifying AC2)
- Catalog auth (`tests/books.test.js`: 2 tests verifying AC3, AC4)
- Catalog read (`tests/books.test.js`: 2 additional tests verifying AC5)
- Cart operations (`tests/cart.test.js`: 4 tests verifying AC6, AC7, AC8, AC9)
- Checkout flows (`tests/checkout.test.js`: 2 tests verifying AC10, AC11)
- Shipping auth (`tests/orders.test.js`: 2 tests verifying AC12, AC13)
- Persistence (`tests/persistence.test.js`: 1 test verifying AC14)

**Total: 14 behavioral tests verifying ACs 1–14.** AC15 is a delivery gate requiring those 14 tests to exist and pass; it is not itself a separate test case.

---

## Technical Notes

- **Schema:** SQLite tables `product` (id, title, author, price, stock), `cart` (id), `cart_item` (cart_id, book_id, quantity), `order` (id, cart_id, status, total, created_at), and `order_item` (order_id, book_id, quantity, unit_price). Create on startup if missing. File path: `data/paperboat.sqlite` per ticket AC1. (Note: repo-context/Book.md specifies `data/booknest.sqlite`; reconcile before deployment or update repo-context to match ticket.)
- **Auth:** `x-role` header (`admin` or `shopper`) controls access to `POST /api/books` and `POST /api/orders/:id/ship`. Requests without `x-role` default to `shopper`.
- **Cart session:** `x-cart-id` header identifies the cart. Generate a new UUID cart id on first `POST /api/cart/items` when none is provided; return it in the response so the client can include it on subsequent cart requests.
- **Checkout transaction:** Atomically validate stock for all cart items, reduce stock for each, create the order and order_item rows, and clear the cart. Roll back on any failure (e.g., insufficient stock, database error).
- **Errors:** Return `{ "error": "<message>" }` with appropriate HTTP status (400 for validation/business rule failures, 403 for authorization failures, 404 for not found, 500 for unexpected server errors).
- **Test files:** All test files live in `tests/` at the repository root. Test setup creates the database before each test suite and clears all tables before each test case to ensure isolation.

---

## Verification

Each acceptance criterion maps to one or more test cases in the test suite. The LLD (`stories/01-Book.md`) details the test structure and assertions. AC1 is verified by the scaffold tests (server startup, database creation); ACs 2–14 are verified by the 14 behavioral tests enumerated in AC15. The test suite (`npm test`) must pass as the delivery gate for this story.
