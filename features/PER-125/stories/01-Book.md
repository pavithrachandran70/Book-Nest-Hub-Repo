# Low-Level Design — PER-125

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

*(Derived from ticket ACs 1–15. AC15 "test suite passes" is a delivery gate fulfilled by the presence of the 14 behavioral tests below; it is not a separate test case.)*

### AC1: Scaffold and Database Creation

**Given** a fresh clone of the repository  
**When** I run `npm install` and `npm start`  
**Then** the API listens on port 3000 and creates `data/paperboat.sqlite` with tables `product`, `cart`, `cart_item`, `order`, and `order_item`

**Note:** Ticket AC1 specifies `data/paperboat.sqlite`. The repo-context/Book.md specifies `data/booknest.sqlite`. This LLD follows the ticket (the authoritative source for this task). Implementations must reconcile this naming conflict: either update the repo-context baseline to match `paperboat.sqlite`, or update the ticket and this LLD to match `booknest.sqlite` before deployment.

---

### AC2: Health Check

**Given** the server is running  
**When** I `GET /api/health`  
**Then** the response is 200 with body `{ "status": "ok" }`

---

### AC3: Admin Can Create Book

**Given** a request with header `x-role: admin` and JSON body `{ "title": "...", "author": "...", "price": 12.99, "stock": 10 }`  
**When** I `POST /api/books`  
**Then** the response is 201 with `{ "id": <book-id>, "title": "...", "author": "...", "price": 12.99, "stock": 10 }`  
**And** `GET /api/books/<book-id>` returns the same book

---

### AC4: Shopper Cannot Create Book

**Given** a request with header `x-role: shopper`  
**When** I `POST /api/books`  
**Then** the response is 403 with `{ "error": "Forbidden" }`

---

### AC5: List Books

**Given** books exist in the catalog  
**When** any role performs `GET /api/books`  
**Then** the response is 200 with a JSON array `[{ "id": 1, "title": "...", "author": "...", "price": 12.99, "stock": 10 }, ...]`

---

### AC6: Add Item to New Cart

**Given** a book exists with `stock: 5`  
**When** a shopper performs `POST /api/cart/items` with `{ "bookId": 1, "quantity": 2 }` and no `x-cart-id` header  
**Then** the response is 201 with a body containing a cart id  
**And** `GET /api/cart` with header `x-cart-id: <that-id>` returns the cart with one line showing the book, quantity 2, unit price, and line total

---

### AC7: Increment Existing Cart Line

**Given** a cart with `x-cart-id: <uuid>` already contains book 1 with quantity 2  
**When** I `POST /api/cart/items` with `{ "bookId": 1, "quantity": 1 }` and header `x-cart-id: <uuid>`  
**Then** the line quantity becomes 3

---

### AC8: Reject Add Above Stock

**Given** a book with `stock: 5` and a cart line with quantity 2  
**When** I `POST /api/cart/items` with `{ "bookId": 1, "quantity": 4 }`  
**Then** the response is 400 with `{ "error": "Insufficient stock" }`  
**And** the cart line quantity remains 2

---

### AC9: Remove Cart Item

**Given** a cart contains a line for book 1  
**When** I `DELETE /api/cart/items/1` with header `x-cart-id: <uuid>`  
**Then** the response is 204  
**And** `GET /api/cart` shows that line removed

---

### AC10: Successful Checkout

**Given** a cart with items whose total quantities are within current stock  
**When** I `POST /api/checkout` with header `x-cart-id: <uuid>`  
**Then** the response is 201 with an order id, status `placed`, the same lines, and the order total  
**And** `GET /api/cart` with the same `x-cart-id` returns an empty cart  
**And** each book's stock is reduced by the ordered quantity

---

### AC11: Checkout Fails on Insufficient Stock

**Given** a cart with a line quantity exceeding current stock  
**When** I `POST /api/checkout`  
**Then** the response is 400  
**And** no order is created  
**And** stock levels remain unchanged

---

### AC12: Admin Can Ship Order

**Given** an order exists with id 1 and status `placed`  
**When** I `POST /api/orders/1/ship` with header `x-role: admin`  
**Then** the order status becomes `shipped`

---

### AC13: Shopper Cannot Ship Order

**Given** the same order  
**When** I `POST /api/orders/1/ship` with header `x-role: shopper`  
**Then** the response is 403

---

### AC14: Data Persists After Restart

**Given** the server has created book 1, a cart, and order 1  
**When** the server restarts  
**Then** `GET /api/books/1` returns the book  
**And** the order is still readable  
**And** the book's stock reflects the reduction from order 1

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
| `src/routes/orders.js` | `POST /api/orders/:id/ship` (admin) |

---

### Test Files

| File | Purpose | Test Count |
|------|---------|------------|
| `tests/health.test.js` | Health endpoint (AC2) | 1 |
| `tests/books.test.js` | Catalog auth and read (AC3, AC4, AC5) | 4 |
| `tests/cart.test.js` | Cart operations (AC6, AC7, AC8, AC9) | 4 |
| `tests/checkout.test.js` | Checkout flows (AC10, AC11) | 2 |
| `tests/orders.test.js` | Shipping auth (AC12, AC13) | 2 |
| `tests/persistence.test.js` | Persistence after restart (AC14) | 1 |

**Total:** 14 behavioral tests verifying ticket ACs 1–14. Ticket AC15 ("run `npm test` and all tests pass") is a delivery gate fulfilled by these 14 tests, not a separate test case.

---

### Data Directory

| Path | Purpose |
|------|---------|
| `data/.gitkeep` | Ensure `data/` exists; `.sqlite` files are gitignored |

---

## Database Schema

**File:** `data/paperboat.sqlite`

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

**Note:** Ticket AC1 specifies "product, cart, cart-item, and order tables." The ticket's wording "order tables" (plural) is interpreted as both the `order` header table and the `order_item` line-item table, which are required to store orders with multiple line items. This five-table design (`product`, `cart`, `cart_item`, `order`, `order_item`) is the minimal implementation satisfying the ticket's intent.

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
router.post('/api/orders/:id/ship', requireAdmin, (req, res) => { /* ship order */ });
```

- **Algorithm:**
  1. Validate order exists with `id = :id`.
  2. If not found: 404 `{ "error": "Order not found" }`.
  3. Update `order` set `status = 'shipped'`.
  4. Return 200 with `{ "orderId": ..., "status": "shipped" }`.

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
  - `beforeAll`: Start server on a test port (e.g., 3001), create `data/paperboat.sqlite`.
  - `beforeEach`: Clear all tables (`DELETE FROM product; DELETE FROM cart; DELETE FROM cart_item; DELETE FROM "order"; DELETE FROM order_item;`).
  - `afterAll`: Stop server, optionally remove test database.

### Test Cases

*(14 behavioral test cases mapping to ticket ACs 1–14. Ticket AC15 "test suite passes" is fulfilled by the existence and passing of these 14 tests, not by a separate test case.)*

#### `tests/health.test.js` (1 test)

1. **Health Check (AC2)**
   - **Given:** Server is running
   - **When:** `GET /api/health`
   - **Then:** Response is 200 with `{ "status": "ok" }`

---

#### `tests/books.test.js` (4 tests)

1. **Admin Create Book (AC3)**
   - **Given:** Request with header `x-role: admin` and body `{ "title": "The Hobbit", "author": "J.R.R. Tolkien", "price": 12.99, "stock": 10 }`
   - **When:** `POST /api/books`
   - **Then:** Response is 201 with book object including id
   - **And:** `GET /api/books/:id` returns the same book

2. **Shopper Cannot Create Book (AC4)**
   - **Given:** Request with header `x-role: shopper`
   - **When:** `POST /api/books`
   - **Then:** Response is 403 with `{ "error": "Forbidden" }`

3. **List All Books (AC5)**
   - **Given:** Books exist in catalog
   - **When:** Any role performs `GET /api/books`
   - **Then:** Response is 200 with array of books including id, title, author, price, stock

4. **Get Book By ID (AC5 continuation)**
   - **Given:** Book exists with id 1
   - **When:** Any role performs `GET /api/books/1`
   - **Then:** Response is 200 with book object

---

#### `tests/cart.test.js` (4 tests)

1. **Add Item to New Cart (AC6)**
   - **Given:** Book exists with stock 5
   - **When:** `POST /api/cart/items` with `{ "bookId": 1, "quantity": 2 }` and no `x-cart-id` header
   - **Then:** Response is 201 with `{ "cartId": "<uuid>", "item": { ... } }`
   - **And:** `GET /api/cart` with header `x-cart-id: <uuid>` shows the item

2. **Increment Existing Cart Line (AC7)**
   - **Given:** Cart contains book 1 with quantity 2
   - **When:** `POST /api/cart/items` with `{ "bookId": 1, "quantity": 1 }` and header `x-cart-id: <uuid>`
   - **Then:** Line quantity becomes 3
   - **And:** `GET /api/cart` shows quantity 3 and updated line total

3. **Reject Add Above Stock (AC8)**
   - **Given:** Book with stock 5 and cart line with quantity 2
   - **When:** `POST /api/cart/items` with `{ "bookId": 1, "quantity": 4 }`
   - **Then:** Response is 400 with `{ "error": "Insufficient stock" }`
   - **And:** Cart line quantity remains 2

4. **Delete Cart Item (AC9)**
   - **Given:** Cart contains line for book 1
   - **When:** `DELETE /api/cart/items/1` with header `x-cart-id: <uuid>`
   - **Then:** Response is 204
   - **And:** `GET /api/cart` shows line removed

---

#### `tests/checkout.test.js` (2 tests)

1. **Successful Checkout (AC10)**
   - **Given:** Cart with items within stock limits
   - **When:** `POST /api/checkout` with header `x-cart-id: <uuid>`
   - **Then:** Response is 201 with order id, status `placed`, items, and total
   - **And:** `GET /api/cart` returns empty cart
   - **And:** Book stock is reduced by ordered quantity

2. **Checkout Fails on Insufficient Stock (AC11)**
   - **Given:** Cart with quantity exceeding current stock
   - **When:** `POST /api/checkout`
   - **Then:** Response is 400
   - **And:** No order is created
   - **And:** Stock levels remain unchanged

---

#### `tests/orders.test.js` (2 tests)

1. **Admin Ships Order (AC12)**
   - **Given:** Order exists with id 1 and status `placed`
   - **When:** `POST /api/orders/1/ship` with header `x-role: admin`
   - **Then:** Response is 200 with `{ "orderId": 1, "status": "shipped" }`

2. **Shopper Cannot Ship Order (AC13)**
   - **Given:** Order exists with id 1 and status `placed`
   - **When:** `POST /api/orders/1/ship` with header `x-role: shopper`
   - **Then:** Response is 403 with `{ "error": "Forbidden" }`

---

#### `tests/persistence.test.js` (1 test)

1. **Data Persists After Restart (AC14)**
   - **Given:** Server has created book 1, a cart, and order 1
   - **When:** Server restarts
   - **Then:** `GET /api/books/1` returns the book
   - **And:** `GET /api/orders/1` returns the order
   - **And:** Book stock reflects the reduction from order 1

---

**Total test count:** 14 behavioral tests verifying ticket ACs 1–14. Ticket AC15 ("run `npm test` and all tests pass") is a delivery gate fulfilled by these 14 tests, not a separate test case.

---

## Implementation Sequence

1. **Scaffold:**
   - `package.json`: dependencies (express, better-sqlite3, jest), scripts (`"start": "node src/server.js"`, `"test": "jest"`).
   - `.gitignore`: `node_modules/`, `data/*.sqlite`.
   - `data/.gitkeep`: ensure directory exists.

2. **Database:**
   - `src/db.js`: open `data/paperboat.sqlite`, run schema DDL, export `db` instance.

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
   - Run `npm start` → verify server starts and creates `data/paperboat.sqlite`.
   - Run `npm test` → verify all 14 tests pass (fulfilling ticket AC15).

---

## Open Questions

- **Database file name conflict:** Ticket AC1 specifies `data/paperboat.sqlite`. The repo-context/Book.md specifies `data/booknest.sqlite`. This LLD follows the ticket (the authoritative source for this task). Before deployment, either update the repo-context baseline to match `paperboat.sqlite`, or update the ticket and this LLD to match `booknest.sqlite`.

---

## Notes

- **Port:** Read from `process.env.PORT || 3000`.
- **Cart ID:** Use `crypto.randomUUID()` (Node.js 20 built-in; no external library).
- **Database file:** `data/paperboat.sqlite` per ticket AC1. (See Open Questions regarding repo-context naming conflict.)
- **Database tables:** Five tables per ticket AC1 and schema design: `product`, `cart`, `cart_item`, `order`, `order_item`. The ticket's wording "order tables" is interpreted as the `order` and `order_item` tables required to store orders with line items.
- **Error messages:** Match ticket ACs exactly (e.g., `"Forbidden"`, `"Insufficient stock"`, `"Insufficient stock for book <id>"`).
- **Order status:** Only `placed` and `shipped` are valid.
- **No TypeScript:** Use plain JavaScript (CommonJS).
- **No frontend:** No HTML, React, or UI files.
- **Test count:** 14 behavioral tests verify ticket ACs 1–14; ticket AC15 ("run `npm test` and all tests pass") is a delivery gate fulfilled by those 14 tests, not a separate test case.
