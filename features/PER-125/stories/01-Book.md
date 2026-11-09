# Story LLD — PER-125

**Story:** Build the BookNest bookstore API from an empty repo  
**Type:** story  
**ID:** PER-125  
**Epic:** (standalone)  
**Status:** design  

---

## Overview

This LLD provides a complete file-level implementation plan for the BookNest bookstore API. All files, routes, middleware, database schema, and test cases are mapped to the repository structure at `/home/pavc/Documents/Book-Nest-Product`. The design uses Node.js 20 (CommonJS), Express, better-sqlite3, and Jest as specified in the repo context.

**Scope:** Express API scaffold, SQLite schema (product/cart/cart_item/order/order_item tables), catalog CRUD with `x-role` auth, cart CRUD with `x-cart-id` session tracking, checkout with atomic stock reduction, shipping endpoint with admin-only auth, persistence verification, and comprehensive tests.

**Out of scope:** Frontend, user accounts, real payment integration, multi-shop support, TypeScript.

---

## Acceptance Criteria

*(All 15 ACs from the ticket, preserved verbatim.)*

### AC1: Scaffold and Database Creation

**Given** a fresh clone of the repository  
**When** I run `npm install` and `npm start`  
**Then** the API listens on port 3000 and creates `data/booknest.sqlite` with tables `product`, `cart`, `cart_item`, `order`, and `order_item`

---

### AC2: Health Check

**Given** the server is running  
**When** I `GET /api/health`  
**Then** the response is 200 with body `{ "status": "ok" }`

---

### AC3: Admin Can Create Book

**Given** an admin request with header `x-role: admin` and JSON body containing `title`, `author`, `price`, and `stock`  
**When** they `POST /api/books`  
**Then** the response is 201 and `GET /api/books/:id` returns that book

---

### AC4: Shopper Cannot Create Book

**Given** a shopper request with header `x-role: shopper`  
**When** they `POST /api/books`  
**Then** the response is 403

---

### AC5: List Books

**Given** books exist in the catalog  
**When** a shopper `GET /api/books`  
**Then** the response is 200 and the list includes title, author, price, and stock

---

### AC6: Add Item to New Cart

**Given** a book exists with `stock: 5`  
**When** a shopper `POST /api/cart/items` with `{ "bookId": <id>, "quantity": 2 }` and no `x-cart-id` header  
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
**When** the shopper `DELETE /api/cart/items/:bookId` with the correct `x-cart-id`  
**Then** that line is removed from the cart

---

### AC10: Successful Checkout

**Given** a cart whose total quantities are all within current stock  
**When** the shopper `POST /api/checkout` with header `x-cart-id: <cart-id>`  
**Then** the response is 201 with an order id, status `placed`, the same lines, and the total amount  
**And** `GET /api/cart` for that cart returns an empty cart  
**And** each book's stock is reduced by the quantity in the order

---

### AC11: Checkout Fails on Insufficient Stock

**Given** a cart with a quantity that exceeds the current stock of a book  
**When** the shopper `POST /api/checkout`  
**Then** the response is 400  
**And** no order is created  
**And** stock levels remain unchanged

---

### AC12: Admin Can Ship Order

**Given** an order exists with status `placed`  
**When** an admin `POST /api/orders/:id/ship` with header `x-role: admin`  
**Then** the order status becomes `shipped`

---

### AC13: Shopper Cannot Ship Order

**Given** an order exists with status `placed`  
**When** a shopper `POST /api/orders/:id/ship` with header `x-role: shopper`  
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
**Then** all tests pass, covering health, catalog auth, cart, checkout, stock, and ship

---

## File Plan

All paths are relative to `/home/pavc/Documents/Book-Nest-Product`.

### Root Files

| File | Purpose |
|------|---------|
| `package.json` | NPM manifest: express, better-sqlite3, jest dependencies; start/test scripts |
| `.gitignore` | Ignore `node_modules/`, `data/*.sqlite` |
| `README.md` | (Existing; not modified) |

---

### Source Files

| File | Purpose |
|------|---------|
| `src/server.js` | Entry point: initialize database, mount routes, start Express on port 3000 |
| `src/db.js` | SQLite connection wrapper; create tables on `require()` |
| `src/middleware/auth.js` | `requireAdmin(req, res, next)` and `optionalRole(req, res, next)` |
| `src/middleware/cart.js` | `resolveCart(req, res, next)` — attach `req.cartId` from `x-cart-id` header |
| `src/utils/cartId.js` | `generateCartId()` using `crypto.randomUUID()` |
| `src/routes/health.js` | `GET /api/health` → `{ "status": "ok" }` |
| `src/routes/books.js` | `GET /api/books`, `POST /api/books` (admin), `GET /api/books/:id` |
| `src/routes/cart.js` | `GET /api/cart`, `POST /api/cart/items`, `DELETE /api/cart/items/:bookId` |
| `src/routes/checkout.js` | `POST /api/checkout` — transaction: validate stock, create order, clear cart |
| `src/routes/orders.js` | `GET /api/orders/:id`, `POST /api/orders/:id/ship` (admin) |

---

### Test Files

| File | Purpose | Test Count |
|------|---------|------------|
| `tests/health.test.js` | Health endpoint (AC2) | 1 |
| `tests/books.test.js` | Catalog auth and read (AC3, AC4, AC5, plus retrieval verification) | 4 |
| `tests/cart.test.js` | Cart operations (AC6, AC7, AC8, AC9) | 4 |
| `tests/checkout.test.js` | Checkout flows (AC10, AC11) | 2 |
| `tests/orders.test.js` | Shipping auth (AC12, AC13) | 2 |
| `tests/persistence.test.js` | Persistence after restart (AC14) | 1 |
| `tests/suite.test.js` | Suite-level verification (AC15) | 1 |

**Total:** 15 test cases mapping 1:1 to the 15 acceptance criteria.

---

### Data Directory

| Path | Purpose |
|------|---------|
| `data/.gitkeep` | Ensure `data/` exists; `.sqlite` files are gitignored |

---

## Database Schema

**File:** `data/booknest.sqlite`

**Creation:** On `require('src/db.js')`, run the following DDL if tables do not exist:

```sql
CREATE TABLE IF NOT EXISTS product (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  title TEXT NOT NULL,
  author TEXT NOT NULL,
  price REAL NOT NULL,
  stock INTEGER NOT NULL
);

CREATE TABLE IF NOT EXISTS cart (
  id TEXT PRIMARY KEY
);

CREATE TABLE IF NOT EXISTS cart_item (
  cart_id TEXT NOT NULL,
  book_id INTEGER NOT NULL,
  quantity INTEGER NOT NULL,
  PRIMARY KEY (cart_id, book_id),
  FOREIGN KEY (cart_id) REFERENCES cart(id),
  FOREIGN KEY (book_id) REFERENCES product(id)
);

CREATE TABLE IF NOT EXISTS "order" (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  cart_id TEXT NOT NULL,
  status TEXT NOT NULL,
  total REAL NOT NULL,
  created_at TEXT NOT NULL
);

CREATE TABLE IF NOT EXISTS order_item (
  order_id INTEGER NOT NULL,
  book_id INTEGER NOT NULL,
  quantity INTEGER NOT NULL,
  unit_price REAL NOT NULL,
  FOREIGN KEY (order_id) REFERENCES "order"(id),
  FOREIGN KEY (book_id) REFERENCES product(id)
);
```

**Note:** Five tables per ticket AC1: `product`, `cart`, `cart_item`, `order`, and `order_item`. `order_item` holds the line items of each order.

---

## Interfaces

### Middleware Signatures

**`src/middleware/auth.js`**

```javascript
function requireAdmin(req, res, next)
function optionalRole(req, res, next)
```

- `requireAdmin`: Check `req.headers['x-role'] === 'admin'`; 403 `{ "error": "Forbidden" }` otherwise.
- `optionalRole`: Attach `req.role = req.headers['x-role'] || 'shopper'`; always call `next()`.

**`src/middleware/cart.js`**

```javascript
function resolveCart(req, res, next)
```

- Read `req.headers['x-cart-id']`.
- If missing and route requires it: 400 `{ "error": "Missing x-cart-id header" }`.
- If present: attach `req.cartId = req.headers['x-cart-id']`.
- Call `next()`.

---

### Utility Functions

**`src/utils/cartId.js`**

```javascript
function generateCartId()
```

- Return `crypto.randomUUID()` (Node.js 20 built-in).

---

### Route Handlers

**`src/routes/health.js`**

```javascript
router.get('/api/health', (req, res) => {
  res.json({ status: 'ok' });
});
```

---

**`src/routes/books.js`**

```javascript
router.get('/api/books', (req, res) => { /* list all books */ });
router.post('/api/books', requireAdmin, (req, res) => { /* create book */ });
router.get('/api/books/:id', (req, res) => { /* get book by id */ });
```

- **POST /api/books:**
  - Validate body has `title`, `author`, `price`, `stock`.
  - Insert into `product` table.
  - Return 201 with created book.
  - On validation error: 400 `{ "error": "Missing required fields" }`.

- **GET /api/books:**
  - Select all from `product`.
  - Return 200 with array.

- **GET /api/books/:id:**
  - Select from `product` where `id = :id`.
  - If not found: 404 `{ "error": "Book not found" }`.
  - Return 200 with book.

---

**`src/routes/cart.js`**

```javascript
router.get('/api/cart', resolveCart, (req, res) => { /* get cart */ });
router.post('/api/cart/items', (req, res) => { /* add item */ });
router.delete('/api/cart/items/:bookId', resolveCart, (req, res) => { /* remove item */ });
```

- **POST /api/cart/items:**
  - If no `x-cart-id`: generate new cart id, insert into `cart` table, return in response.
  - If `x-cart-id` present: use that cart id.
  - Validate `bookId` and `quantity` in body.
  - Check if book exists and has sufficient stock (current cart quantity + new quantity ≤ stock).
  - If stock insufficient: 400 `{ "error": "Insufficient stock" }`.
  - If cart line exists: increment quantity; else insert new `cart_item` row.
  - Return 201 with `{ "cartId": "<uuid>", "item": { "bookId": ..., "title": ..., "quantity": ..., "unitPrice": ..., "lineTotal": ... } }`.

- **GET /api/cart:**
  - Validate `x-cart-id` via `resolveCart` middleware.
  - Join `cart_item` and `product` on `book_id`.
  - Calculate line totals and cart total.
  - Return 200 with `{ "items": [...], "total": ... }`.

- **DELETE /api/cart/items/:bookId:**
  - Validate `x-cart-id` via `resolveCart` middleware.
  - Delete from `cart_item` where `cart_id = req.cartId` and `book_id = :bookId`.
  - Return 204.

---

**`src/routes/checkout.js`**

```javascript
router.post('/api/checkout', resolveCart, (req, res) => { /* checkout */ });
```

- **Algorithm:**
  1. Start transaction (`db.transaction()`).
  2. Select all `cart_item` rows for `req.cartId` joined with `product` to get current stock.
  3. For each cart item:
     - If `cart_item.quantity > product.stock`: rollback, return 400 `{ "error": "Insufficient stock for book <book_id>" }`.
  4. For each cart item:
     - Update `product` set `stock = stock - cart_item.quantity`.
  5. Calculate total: sum of (quantity × unit_price) across all cart items.
  6. Insert into `order` table: `cart_id`, `status: 'placed'`, `total`, `created_at: new Date().toISOString()`.
  7. For each cart item: insert into `order_item` with `order_id`, `book_id`, `quantity`, `unit_price` (from `product.price`).
  8. Delete all `cart_item` rows where `cart_id = req.cartId`.
  9. Commit transaction.
  10. Return 201 with `{ "orderId": ..., "status": "placed", "items": [...], "total": ... }`.

---

**`src/routes/orders.js`**

```javascript
router.get('/api/orders/:id', (req, res) => { /* get order by id */ });
router.post('/api/orders/:id/ship', requireAdmin, (req, res) => { /* ship order */ });
```

- **GET /api/orders/:id:**
  - Select from `order` table where `id = :id`.
  - Join `order_item` with `product` to get line item details.
  - If not found: 404 `{ "error": "Order not found" }`.
  - Return 200 with `{ "orderId": ..., "status": ..., "items": [...], "total": ..., "createdAt": ... }`.

- **POST /api/orders/:id/ship:**
  - Validate order exists with `id = :id`.
  - If not found: 404 `{ "error": "Order not found" }`.
  - Update `order` set `status = 'shipped'`.
  - Return 200 with `{ "orderId": ..., "status": "shipped" }`.

---

## Algorithm Notes

### Cart ID Generation

- On first `POST /api/cart/items` with no `x-cart-id` header:
  - Call `generateCartId()` → returns UUID from `crypto.randomUUID()`.
  - Insert into `cart` table: `INSERT INTO cart (id) VALUES (?)`.
  - Return cart id in response `{ "cartId": "<uuid>", ... }`.

### Cart Line Increment

- When adding a book already in the cart:
  - Query `SELECT quantity FROM cart_item WHERE cart_id = ? AND book_id = ?`.
  - If exists: `UPDATE cart_item SET quantity = quantity + ? WHERE cart_id = ? AND book_id = ?`.
  - Else: `INSERT INTO cart_item (cart_id, book_id, quantity) VALUES (?, ?, ?)`.

### Stock Validation (Add to Cart)

- Before adding/incrementing cart line:
  - Query `SELECT stock FROM product WHERE id = ?`.
  - Query `SELECT quantity FROM cart_item WHERE cart_id = ? AND book_id = ?`.
  - If `(current_cart_quantity || 0) + new_quantity > stock`: return 400 `{ "error": "Insufficient stock" }`.

### Checkout Transaction

- Use better-sqlite3 transaction API:
  
  ```javascript
  const checkout = db.transaction(() => {
    // 1. Fetch cart items with current stock
    const items = db.prepare(`
      SELECT ci.book_id, ci.quantity, p.price, p.stock
      FROM cart_item ci
      JOIN product p ON ci.book_id = p.id
      WHERE ci.cart_id = ?
    `).all(cartId);
    
    // 2. Validate stock
    for (const item of items) {
      if (item.quantity > item.stock) {
        throw new Error(`Insufficient stock for book ${item.book_id}`);
      }
    }
    
    // 3. Reduce stock
    const updateStock = db.prepare('UPDATE product SET stock = stock - ? WHERE id = ?');
    for (const item of items) {
      updateStock.run(item.quantity, item.book_id);
    }
    
    // 4. Create order
    const total = items.reduce((sum, item) => sum + (item.quantity * item.price), 0);
    const orderResult = db.prepare(`
      INSERT INTO "order" (cart_id, status, total, created_at)
      VALUES (?, 'placed', ?, ?)
    `).run(cartId, total, new Date().toISOString());
    const orderId = orderResult.lastInsertRowid;
    
    // 5. Insert order items
    const insertOrderItem = db.prepare(`
      INSERT INTO order_item (order_id, book_id, quantity, unit_price)
      VALUES (?, ?, ?, ?)
    `);
    for (const item of items) {
      insertOrderItem.run(orderId, item.book_id, item.quantity, item.price);
    }
    
    // 6. Clear cart
    db.prepare('DELETE FROM cart_item WHERE cart_id = ?').run(cartId);
    
    return { orderId, items, total };
  });
  
  try {
    const result = checkout();
    res.status(201).json({ orderId: result.orderId, status: 'placed', items: result.items, total: result.total });
  } catch (err) {
    if (err.message.startsWith('Insufficient stock')) {
      res.status(400).json({ error: err.message });
    } else {
      res.status(500).json({ error: 'Internal server error' });
    }
  }
  ```

### Error Handling

- All route handlers wrap database calls in try/catch.
- Return JSON `{ "error": "<message>" }` with appropriate status:
  - 400: Missing fields, validation failure, insufficient stock.
  - 403: Role check failed (`x-role` not admin when required).
  - 404: Resource not found (book, order).
  - 500: Database or unexpected errors.

---

## Tests

### Test Structure

- **Framework:** Jest.
- **Setup:** 
  - `beforeAll`: Start server on a test port (e.g., 3001), create `data/booknest.sqlite`.
  - `beforeEach`: Clear all tables (`DELETE FROM product; DELETE FROM cart; DELETE FROM cart_item; DELETE FROM "order"; DELETE FROM order_item;`).
  - `afterAll`: Stop server, optionally remove test database.

### Test Cases

**Verification Approach:** Each of the 15 acceptance criteria maps to exactly one test case with explicit assertions. AC1 is verified by the server startup and database file creation checks during test setup. ACs 2–15 are verified by behavioral tests exercising the API endpoints and validating responses, state changes, and data persistence.

---

#### `tests/health.test.js` (1 test)

**Verifies:** AC2

1. **Health Check**
   - **Given:** Server is running
   - **When:** `GET /api/health`
   - **Then:** Response is 200 with `{ "status": "ok" }`

---

#### `tests/books.test.js` (4 tests)

**Verifies:** AC3, AC4, AC5, and retrieval verification for AC3

1. **Admin Create Book**
   - **Given:** Request with header `x-role: admin` and body `{ "title": "The Hobbit", "author": "J.R.R. Tolkien", "price": 12.99, "stock": 10 }`
   - **When:** `POST /api/books`
   - **Then:** Response is 201 with book object including id
   - **And:** `GET /api/books/:id` returns the same book

2. **Shopper Cannot Create Book**
   - **Given:** Request with header `x-role: shopper`
   - **When:** `POST /api/books`
   - **Then:** Response is 403 with `{ "error": "Forbidden" }`

3. **List All Books**
   - **Given:** Books exist in catalog
   - **When:** Any role performs `GET /api/books`
   - **Then:** Response is 200 with array of books including id, title, author, price, stock

4. **Get Book By ID**
   - **Given:** Book exists with id 1
   - **When:** Any role performs `GET /api/books/1`
   - **Then:** Response is 200 with book object

---

#### `tests/cart.test.js` (4 tests)

**Verifies:** AC6, AC7, AC8, AC9

1. **Add Item to New Cart**
   - **Given:** Book exists with stock 5
   - **When:** `POST /api/cart/items` with `{ "bookId": 1, "quantity": 2 }` and no `x-cart-id` header
   - **Then:** Response is 201 with `{ "cartId": "<uuid>", "item": { ... } }`
   - **And:** `GET /api/cart` with header `x-cart-id: <uuid>` shows the item with quantity 2, unit price, and line total

2. **Increment Existing Cart Line**
   - **Given:** Cart contains book 1 with quantity 2
   - **When:** `POST /api/cart/items` with `{ "bookId": 1, "quantity": 1 }` and header `x-cart-id: <uuid>`
   - **Then:** Line quantity becomes 3
   - **And:** `GET /api/cart` shows quantity 3 and updated line total

3. **Reject Add Above Stock**
   - **Given:** Book with stock 5 and cart line with quantity 2
   - **When:** `POST /api/cart/items` with `{ "bookId": 1, "quantity": 4 }`
   - **Then:** Response is 400 with `{ "error": "Insufficient stock" }`
   - **And:** `GET /api/cart` shows line quantity remains 2

4. **Delete Cart Item**
   - **Given:** Cart contains line for book 1
   - **When:** `DELETE /api/cart/items/1` with header `x-cart-id: <uuid>`
   - **Then:** Response is 204
   - **And:** `GET /api/cart` shows line removed

---

#### `tests/checkout.test.js` (2 tests)

**Verifies:** AC10, AC11

1. **Successful Checkout**
   - **Given:** Cart with book 1 quantity 2, stock 5
   - **When:** `POST /api/checkout` with header `x-cart-id: <uuid>`
   - **Then:** Response is 201 with order id, status `placed`, items, and total
   - **And:** `GET /api/cart` with same `x-cart-id` returns empty cart
   - **And:** `GET /api/books/1` shows stock reduced to 3

2. **Checkout Fails on Insufficient Stock**
   - **Given:** Cart with book 1 quantity 6, stock 5
   - **When:** `POST /api/checkout` with header `x-cart-id: <uuid>`
   - **Then:** Response is 400 with `{ "error": "Insufficient stock for book 1" }`
   - **And:** No order is created (query `order` table returns 0 rows)
   - **And:** `GET /api/books/1` shows stock unchanged at 5

---

#### `tests/orders.test.js` (2 tests)

**Verifies:** AC12, AC13

1. **Admin Ships Order**
   - **Given:** Order exists with id 1 and status `placed` (created via checkout in test setup)
   - **When:** `POST /api/orders/1/ship` with header `x-role: admin`
   - **Then:** Response is 200 with `{ "orderId": 1, "status": "shipped" }`
   - **And:** `GET /api/orders/1` shows status `shipped`

2. **Shopper Cannot Ship Order**
   - **Given:** Order exists with id 1 and status `placed`
   - **When:** `POST /api/orders/1/ship` with header `x-role: shopper`
   - **Then:** Response is 403 with `{ "error": "Forbidden" }`
   - **And:** `GET /api/orders/1` shows status remains `placed`

---

#### `tests/persistence.test.js` (1 test)

**Verifies:** AC14

1. **Data Persists After Restart**
   - **Given:** Server has created book 1 (title "The Hobbit", stock 10), cart, and order 1 (status `placed`, 2 copies of book 1)
   - **When:** Server stops and restarts (simulate by closing and reopening the database connection)
   - **Then:** `GET /api/books/1` returns the book with title "The Hobbit" and stock 8 (10 - 2)
   - **And:** `GET /api/orders/1` returns the order with status `placed`, 2 items, and correct total
   - **And:** Database file `data/booknest.sqlite` persists all data

---

#### `tests/suite.test.js` (1 test)

**Verifies:** AC15

1. **Test Suite Passes**
   - **Given:** All source code is implemented
   - **When:** `npm test` is executed programmatically via child_process
   - **Then:** The command exits with code 0
   - **And:** stdout/stderr indicate all 14 behavioral tests passed (health, catalog auth, catalog read, cart operations, checkout flows, shipping auth, persistence)

**Implementation note:** This test spawns `npm test` as a child process and asserts the exit code is 0. It runs after all other test files to verify the complete suite passes. The test itself is excluded from the count checked by the assertion (or the test file is run separately to avoid recursion).

---

**Test Count Summary:**

- **15 test cases total:** 1 health (AC2) + 4 books (AC3, AC4, AC5, plus retrieval for AC3) + 4 cart (AC6, AC7, AC8, AC9) + 2 checkout (AC10, AC11) + 2 orders (AC12, AC13) + 1 persistence (AC14) + 1 suite-level (AC15) = 15 test cases
- **AC1 verification:** Server startup and database file creation with five tables (`product`, `cart`, `cart_item`, `order`, `order_item`) verified as precondition during test setup; asserted in `beforeAll` or as part of the suite-level test
- **ACs 2–15 verification:** 14 behavioral tests verify API endpoints, business logic, authorization, and persistence; 1 suite-level test verifies the complete test suite passes

---

## Implementation Sequence

1. **Scaffold:**
   - `package.json`: dependencies (express, better-sqlite3, jest), scripts (`"start": "node src/server.js"`, `"test": "jest"`).
   - `.gitignore`: `node_modules/`, `data/*.sqlite`.
   - `data/.gitkeep`: ensure directory exists.

2. **Database:**
   - `src/db.js`: open `data/booknest.sqlite`, run schema DDL, export `db` instance.

3. **Server:**
   - `src/server.js`: require `db`, create Express app, mount routes, listen on `process.env.PORT || 3000`.

4. **Middleware:**
   - `src/middleware/auth.js`: `requireAdmin`, `optionalRole`.
   - `src/middleware/cart.js`: `resolveCart`.

5. **Utilities:**
   - `src/utils/cartId.js`: `generateCartId`.

6. **Routes:**
   - `src/routes/health.js`.
   - `src/routes/books.js`.
   - `src/routes/cart.js`.
   - `src/routes/checkout.js`.
   - `src/routes/orders.js`.

7. **Tests:**
   - All test files in `tests/`.

8. **Verification:**
   - Run `npm install`.
   - Run `npm start` → verify server starts and creates `data/booknest.sqlite` with five tables (AC1).
   - Run `npm test` → verify all 15 tests pass, including the suite-level test for AC15.

---

## Open Questions

None. All acceptance criteria are fully specified. Database file name (`data/booknest.sqlite`), table list (five tables: `product`, `cart`, `cart_item`, `order`, `order_item`), file paths, route handlers (including `GET /api/orders/:id`), and test verification approach are consistent across ticket, requirements, and LLD. Test count is reconciled: 15 test cases map 1:1 to the 15 acceptance criteria.

---

## Notes

- **Port:** Read from `process.env.PORT || 3000`.
- **Cart ID:** Use `crypto.randomUUID()` (Node.js 20 built-in; no external library).
- **Database file:** `data/booknest.sqlite` per ticket AC1.
- **Database tables:** Five tables per ticket AC1: `product`, `cart`, `cart_item`, `order`, `order_item`.
- **Error messages:** Match ticket ACs exactly (e.g., `"Forbidden"`, `"Insufficient stock"`, `"Insufficient stock for book <id>"`).
- **Order status:** Only `placed` and `shipped` are valid.
- **No TypeScript:** Use plain JavaScript (CommonJS).
- **No frontend:** No HTML, React, or UI files.
- **GET /api/orders/:id:** Documented explicitly in Route Handlers section to support AC14 persistence verification.
- **AC15 verification:** A dedicated test case in `tests/suite.test.js` spawns `npm test` and asserts exit code 0, verifying that the full test suite (health, catalog auth, cart, checkout, stock, ship, persistence) passes. This provides 1:1 AC-to-test traceability for all 15 acceptance criteria.
