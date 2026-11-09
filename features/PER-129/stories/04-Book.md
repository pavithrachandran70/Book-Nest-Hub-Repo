# Test Automation Summary — Story 04-Book

**Story:** Checkout with stock reduction and order shipping  
**Repo:** Book  
**Story ID:** 04-Book  
**Generated:** 2025-06-10

---

## Overview

This document provides the automated test specification for Story 04-Book, which implements the checkout endpoint (POST /api/checkout) that validates cart quantities against stock in a transaction, creates an order with line items, reduces stock, and empties the cart. It also implements the order shipping endpoint (POST /api/orders/:id/ship, admin only) that updates order status to "shipped". All test cases map 1:1 to acceptance criteria and verification steps defined in the breakdown.

---

## Test Coverage Matrix

| AC/Verification Step | Test File | Test Case | Status |
|---------------------|-----------|-----------|--------|
| AC-10 part 1: Successful checkout returns 201 with order | `__tests__/checkout.test.js` | Test 1 | ✓ Implemented |
| AC-10 part 2: Cart is empty after checkout | `__tests__/checkout.test.js` | Test 2 | ✓ Implemented |
| AC-10 part 3: Stock is reduced after checkout | `__tests__/checkout.test.js` | Test 3 | ✓ Implemented |
| AC-10 part 4: order_item captures price at checkout time | `__tests__/checkout.test.js` | Test 4 | ✓ Implemented |
| AC-11 part 1: Checkout with quantity > stock returns 400 | `__tests__/checkout.test.js` | Test 5 | ✓ Implemented |
| AC-11 part 2: Stock unchanged after failed checkout | `__tests__/checkout.test.js` | Test 6 | ✓ Implemented |
| AC-11 part 3: No order created after failed checkout | `__tests__/checkout.test.js` | Test 7 | ✓ Implemented |
| Verification: Checkout with empty cart returns 400 | `__tests__/checkout.test.js` | Test 8 | ✓ Implemented |
| AC-12: Admin ship order returns 200 with status "shipped" | `__tests__/orders.test.js` | Test 1 | ✓ Implemented |
| AC-12 part 2: Order status persists in database | `__tests__/orders.test.js` | Test 2 | ✓ Implemented |
| AC-13: Shopper ship order returns 403 | `__tests__/orders.test.js` | Test 3 | ✓ Implemented |
| Verification: Ship nonexistent order returns 404 | `__tests__/orders.test.js` | Test 4 | ✓ Implemented |
| Verification: Multi-item checkout creates order with multiple items | `__tests__/orders.test.js` | Test 5 | ✓ Implemented |
| AC-14: Persistence across restart (order and stock) | `__tests__/orders.test.js` | Test 6 | ✓ Implemented |

**Total Test Count:** 14 tests (8 checkout + 6 orders, matches breakdown verification approach count exactly)

---

## Tests

### __tests__/checkout.test.js

**Purpose:** Test suite for checkout endpoint and stock reduction.

**Test Framework:** Jest + Supertest

**Test Isolation:**
- `beforeEach`: Delete all rows from `order_item`, `order`, `cart_item`, `cart`, and `product` tables (in foreign key dependency order)
- `afterAll`: Close database connection

**Helper Functions:**
- `createBook(title, author, price, stock)`: Admin POST to create a book, returns the book object
- `addToCart(cartId, bookId, quantity)`: POST `/api/cart/items` with optional cartId, returns response body
- `getCart(cartId)`: GET `/api/cart` with cartId, returns response body

**Test Cases:**

#### Test 1: AC-10 part 1 — Successful checkout returns 201 with order

**Acceptance Criterion:**
> **Given** a cart with quantities that are in stock  
> **When** the shopper POST `/api/checkout` with that `x-cart-id`  
> **Then** the response is 201 with an order object containing `id`, `status: "placed"`, the same line items, and `total`

**Test Implementation:**
```javascript
test('AC-10 part 1: Successful checkout returns 201 with order', async () => {
  const book = await createBook('Test Book', 'Test Author', 10.00, 5);
  const cartData = await addToCart(null, book.id, 2);
  const cartId = cartData.cartId;

  const response = await request(app)
    .post('/api/checkout')
    .set('x-cart-id', cartId.toString());

  expect(response.status).toBe(201);
  expect(response.body).toMatchObject({
    id: expect.any(Number),
    status: 'placed',
    total: 20.00,
    createdAt: expect.any(String)
  });
  expect(response.body.items).toHaveLength(1);
  expect(response.body.items[0]).toMatchObject({
    bookId: book.id,
    quantity: 2,
    price: 10.00
  });
});
```

**Assertions:**
1. HTTP status code is 201
2. Response body includes auto-generated `id` (integer)
3. Response body `status` is "placed"
4. Response body `total` is 20.00 (2 × 10.00)
5. Response body includes `createdAt` timestamp
6. Items array has exactly 1 entry
7. First item has correct `bookId`, `quantity: 2`, `price: 10.00`

**Traceability:**
- Maps to AC-10 part 1 directly
- Maps to breakdown verification step 1: "Checkout with valid cart → 201 with order"

---

#### Test 2: AC-10 part 2 — Cart is empty after checkout

**Acceptance Criterion:**
> **And** GET `/api/cart` for that cart returns an empty cart

**Test Implementation:**
```javascript
test('AC-10 part 2: Cart is empty after checkout', async () => {
  const book = await createBook('Test Book', 'Test Author', 10.00, 5);
  const cartData = await addToCart(null, book.id, 2);
  const cartId = cartData.cartId;

  await request(app)
    .post('/api/checkout')
    .set('x-cart-id', cartId.toString());

  const cartResponse = await request(app)
    .get('/api/cart')
    .set('x-cart-id', cartId.toString());

  expect(cartResponse.status).toBe(200);
  expect(cartResponse.body.items).toHaveLength(0);
});
```

**Assertions:**
1. After checkout, GET /api/cart returns 200
2. Cart items array is empty (length 0)

**Traceability:**
- Maps to AC-10 part 2 directly
- Maps to breakdown verification step 2: "Cart empty after checkout"

---

#### Test 3: AC-10 part 3 — Stock is reduced after checkout

**Acceptance Criterion:**
> **And** each book's stock is reduced by the ordered quantity

**Test Implementation:**
```javascript
test('AC-10 part 3: Stock is reduced after checkout', async () => {
  const book = await createBook('Test Book', 'Test Author', 10.00, 5);
  const cartData = await addToCart(null, book.id, 2);
  const cartId = cartData.cartId;

  await request(app)
    .post('/api/checkout')
    .set('x-cart-id', cartId.toString());

  const bookResponse = await request(app).get(`/api/books/${book.id}`);

  expect(bookResponse.status).toBe(200);
  expect(bookResponse.body.stock).toBe(3);
});
```

**Assertions:**
1. After checkout, GET /api/books/:id returns 200
2. Book stock is reduced from 5 to 3 (5 - 2)

**Traceability:**
- Maps to AC-10 part 3 directly
- Maps to breakdown verification step 3: "Stock reduced by ordered quantity"

---

#### Test 4: AC-10 part 4 — order_item captures price at checkout time

**Verification Step (not a new AC):**
> Query order_item table → price matches book price at checkout time

**Test Implementation:**
```javascript
test('AC-10 part 4: order_item captures price at checkout time', async () => {
  const book = await createBook('Test Book', 'Test Author', 10.00, 5);
  const cartData = await addToCart(null, book.id, 2);
  const cartId = cartData.cartId;

  const checkoutResponse = await request(app)
    .post('/api/checkout')
    .set('x-cart-id', cartId.toString());

  const orderId = checkoutResponse.body.id;

  const orderItem = db.prepare('SELECT quantity, price FROM order_item WHERE order_id = ?').get(orderId);

  expect(orderItem).toMatchObject({
    quantity: 2,
    price: 10.00
  });
});
```

**Assertions:**
1. Direct database query confirms order_item row exists
2. order_item.quantity is 2
3. order_item.price is 10.00 (matches book price at checkout time)

**Traceability:**
- Maps to breakdown verification step 4: "order_item captures price at checkout time"
- Clarifies AC-10 behavior: price is frozen at order time, not linked to product.price

---

#### Test 5: AC-11 part 1 — Checkout with quantity > stock returns 400

**Acceptance Criterion:**
> **Given** a cart line quantity exceeds the book's current stock  
> **When** the shopper POST `/api/checkout`  
> **Then** the response is 400 with `{ "error": "<message>" }`

**Test Implementation:**
```javascript
test('AC-11 part 1: Checkout with quantity > stock returns 400', async () => {
  const book = await createBook('Test Book', 'Test Author', 10.00, 3);
  const cartData = await addToCart(null, book.id, 2);
  const cartId = cartData.cartId;

  // Manually reduce stock to simulate race condition
  db.prepare('UPDATE product SET stock = ? WHERE id = ?').run(1, book.id);

  const response = await request(app)
    .post('/api/checkout')
    .set('x-cart-id', cartId.toString());

  expect(response.status).toBe(400);
  expect(response.body.error).toMatch(/Insufficient stock/);
});
```

**Assertions:**
1. HTTP status code is 400
2. Response body error message matches /Insufficient stock/

**Traceability:**
- Maps to AC-11 part 1 directly
- Maps to breakdown verification step 5: "Checkout with quantity > stock → 400"

---

#### Test 6: AC-11 part 2 — Stock unchanged after failed checkout

**Acceptance Criterion:**
> **And** stock is unchanged

**Test Implementation:**
```javascript
test('AC-11 part 2: Stock unchanged after failed checkout', async () => {
  const book = await createBook('Test Book', 'Test Author', 10.00, 3);
  const cartData = await addToCart(null, book.id, 2);
  const cartId = cartData.cartId;

  db.prepare('UPDATE product SET stock = ? WHERE id = ?').run(1, book.id);

  await request(app)
    .post('/api/checkout')
    .set('x-cart-id', cartId.toString());

  const bookResponse = await request(app).get(`/api/books/${book.id}`);

  expect(bookResponse.status).toBe(200);
  expect(bookResponse.body.stock).toBe(1);
});
```

**Assertions:**
1. After failed checkout, GET /api/books/:id returns 200
2. Stock is still 1 (unchanged from before the failed checkout attempt)

**Traceability:**
- Maps to AC-11 part 2 directly
- Maps to breakdown verification step 6: "Stock unchanged after validation failure"

---

#### Test 7: AC-11 part 3 — No order created after failed checkout

**Acceptance Criterion:**
> **And** no order is created

**Test Implementation:**
```javascript
test('AC-11 part 3: No order created after failed checkout', async () => {
  const book = await createBook('Test Book', 'Test Author', 10.00, 3);
  const cartData = await addToCart(null, book.id, 2);
  const cartId = cartData.cartId;

  db.prepare('UPDATE product SET stock = ? WHERE id = ?').run(1, book.id);

  await request(app)
    .post('/api/checkout')
    .set('x-cart-id', cartId.toString());

  const orderCount = db.prepare('SELECT COUNT(*) as count FROM "order"').get().count;

  expect(orderCount).toBe(0);
});
```

**Assertions:**
1. Direct database query confirms no order row was created (count is 0)

**Traceability:**
- Maps to AC-11 part 3 directly
- Maps to breakdown verification step 7: "No order created after failed checkout"

---

#### Test 8: Checkout with empty cart returns 400

**Verification Step (not a new AC):**
> POST `/api/checkout` with empty cart → 400 "Cart is empty"

**Test Implementation:**
```javascript
test('Checkout with empty cart returns 400', async () => {
  const book = await createBook('Test Book', 'Test Author', 10.00, 5);
  const cartData = await addToCart(null, book.id, 1);
  const cartId = cartData.cartId;

  // Delete the cart item
  await request(app)
    .delete(`/api/cart/items/${book.id}`)
    .set('x-cart-id', cartId.toString());

  const response = await request(app)
    .post('/api/checkout')
    .set('x-cart-id', cartId.toString());

  expect(response.status).toBe(400);
  expect(response.body).toEqual({ error: 'Cart is empty' });
});
```

**Assertions:**
1. HTTP status code is 400
2. Response body is exactly `{ "error": "Cart is empty" }`

**Traceability:**
- Maps to breakdown verification step 8: "Checkout with empty cart → 400"
- Validates REQ-7 validation rules (cart must contain at least one item)

---

## Complete Test File: __tests__/checkout.test.js

```javascript
const request = require('supertest');
const app = require('../src/app');
const { db } = require('../src/db');

describe('Checkout endpoint and stock reduction', () => {
  async function createBook(title, author, price, stock) {
    const response = await request(app)
      .post('/api/books')
      .set('x-role', 'admin')
      .send({ title, author, price, stock });
    return response.body;
  }

  async function addToCart(cartId, bookId, quantity) {
    const req = request(app).post('/api/cart/items').send({ bookId, quantity });
    if (cartId) {
      req.set('x-cart-id', cartId.toString());
    }
    const response = await req;
    return response.body;
  }

  async function getCart(cartId) {
    const response = await request(app)
      .get('/api/cart')
      .set('x-cart-id', cartId.toString());
    return response.body;
  }

  beforeEach(() => {
    db.prepare('DELETE FROM order_item').run();
    db.prepare('DELETE FROM "order"').run();
    db.prepare('DELETE FROM cart_item').run();
    db.prepare('DELETE FROM cart').run();
    db.prepare('DELETE FROM product').run();
  });

  afterAll(() => {
    db.close();
  });

  test('AC-10 part 1: Successful checkout returns 201 with order', async () => {
    const book = await createBook('Test Book', 'Test Author', 10.00, 5);
    const cartData = await addToCart(null, book.id, 2);
    const cartId = cartData.cartId;

    const response = await request(app)
      .post('/api/checkout')
      .set('x-cart-id', cartId.toString());

    expect(response.status).toBe(201);
    expect(response.body).toMatchObject({
      id: expect.any(Number),
      status: 'placed',
      total: 20.00,
      createdAt: expect.any(String)
    });
    expect(response.body.items).toHaveLength(1);
    expect(response.body.items[0]).toMatchObject({
      bookId: book.id,
      quantity: 2,
      price: 10.00
    });
  });

  test('AC-10 part 2: Cart is empty after checkout', async () => {
    const book = await createBook('Test Book', 'Test Author', 10.00, 5);
    const cartData = await addToCart(null, book.id, 2);
    const cartId = cartData.cartId;

    await request(app)
      .post('/api/checkout')
      .set('x-cart-id', cartId.toString());

    const cartResponse = await request(app)
      .get('/api/cart')
      .set('x-cart-id', cartId.toString());

    expect(cartResponse.status).toBe(200);
    expect(cartResponse.body.items).toHaveLength(0);
  });

  test('AC-10 part 3: Stock is reduced after checkout', async () => {
    const book = await createBook('Test Book', 'Test Author', 10.00, 5);
    const cartData = await addToCart(null, book.id, 2);
    const cartId = cartData.cartId;

    await request(app)
      .post('/api/checkout')
      .set('x-cart-id', cartId.toString());

    const bookResponse = await request(app).get(`/api/books/${book.id}`);

    expect(bookResponse.status).toBe(200);
    expect(bookResponse.body.stock).toBe(3);
  });

  test('AC-10 part 4: order_item captures price at checkout time', async () => {
    const book = await createBook('Test Book', 'Test Author', 10.00, 5);
    const cartData = await addToCart(null, book.id, 2);
    const cartId = cartData.cartId;

    const checkoutResponse = await request(app)
      .post('/api/checkout')
      .set('x-cart-id', cartId.toString());

    const orderId = checkoutResponse.body.id;

    const orderItem = db.prepare('SELECT quantity, price FROM order_item WHERE order_id = ?').get(orderId);

    expect(orderItem).toMatchObject({
      quantity: 2,
      price: 10.00
    });
  });

  test('AC-11 part 1: Checkout with quantity > stock returns 400', async () => {
    const book = await createBook('Test Book', 'Test Author', 10.00, 3);
    const cartData = await addToCart(null, book.id, 2);
    const cartId = cartData.cartId;

    // Manually reduce stock to simulate race condition
    db.prepare('UPDATE product SET stock = ? WHERE id = ?').run(1, book.id);

    const response = await request(app)
      .post('/api/checkout')
      .set('x-cart-id', cartId.toString());

    expect(response.status).toBe(400);
    expect(response.body.error).toMatch(/Insufficient stock/);
  });

  test('AC-11 part 2: Stock unchanged after failed checkout', async () => {
    const book = await createBook('Test Book', 'Test Author', 10.00, 3);
    const cartData = await addToCart(null, book.id, 2);
    const cartId = cartData.cartId;

    db.prepare('UPDATE product SET stock = ? WHERE id = ?').run(1, book.id);

    await request(app)
      .post('/api/checkout')
      .set('x-cart-id', cartId.toString());

    const bookResponse = await request(app).get(`/api/books/${book.id}`);

    expect(bookResponse.status).toBe(200);
    expect(bookResponse.body.stock).toBe(1);
  });

  test('AC-11 part 3: No order created after failed checkout', async () => {
    const book = await createBook('Test Book', 'Test Author', 10.00, 3);
    const cartData = await addToCart(null, book.id, 2);
    const cartId = cartData.cartId;

    db.prepare('UPDATE product SET stock = ? WHERE id = ?').run(1, book.id);

    await request(app)
      .post('/api/checkout')
      .set('x-cart-id', cartId.toString());

    const orderCount = db.prepare('SELECT COUNT(*) as count FROM "order"').get().count;

    expect(orderCount).toBe(0);
  });

  test('Checkout with empty cart returns 400', async () => {
    const book = await createBook('Test Book', 'Test Author', 10.00, 5);
    const cartData = await addToCart(null, book.id, 1);
    const cartId = cartData.cartId;

    // Delete the cart item
    await request(app)
      .delete(`/api/cart/items/${book.id}`)
      .set('x-cart-id', cartId.toString());

    const response = await request(app)
      .post('/api/checkout')
      .set('x-cart-id', cartId.toString());

    expect(response.status).toBe(400);
    expect(response.body).toEqual({ error: 'Cart is empty' });
  });
});
```

---

## Tests

### __tests__/orders.test.js

**Purpose:** Test suite for order shipping endpoint and role enforcement.

**Test Framework:** Jest + Supertest

**Test Isolation:**
- `beforeEach`: Delete all rows from `order_item`, `order`, `cart_item`, `cart`, and `product` tables
- `afterAll`: Close database connection

**Helper Functions:**
- `createBook(title, author, price, stock)`: Admin POST to create a book
- `createOrder(bookId, quantity, price)`: Direct database INSERT to create an order with one item (bypasses checkout for test isolation)

**Test Cases:**

#### Test 1: AC-12 — Admin ship order returns 200 with status "shipped"

**Acceptance Criterion:**
> **Given** an order with status "placed"  
> **When** an admin POST `/api/orders/:id/ship` with `x-role: admin`  
> **Then** the response is 200 and the order status becomes "shipped"

**Test Implementation:**
```javascript
test('AC-12: Admin ship order returns 200 with status "shipped"', async () => {
  const book = await createBook('Test Book', 'Test Author', 10.00, 5);
  const orderId = createOrder(book.id, 2, 10.00);

  const response = await request(app)
    .post(`/api/orders/${orderId}/ship`)
    .set('x-role', 'admin');

  expect(response.status).toBe(200);
  expect(response.body).toMatchObject({
    id: orderId,
    status: 'shipped',
    total: 20.00,
    createdAt: expect.any(String)
  });
});
```

**Assertions:**
1. HTTP status code is 200
2. Response body includes order `id`
3. Response body `status` is "shipped"
4. Response body `total` is 20.00
5. Response body includes `createdAt` timestamp

**Traceability:**
- Maps to AC-12 directly
- Maps to breakdown verification step 9: "Admin ship order → status shipped"

---

#### Test 2: AC-12 part 2 — Order status persists in database

**Verification Step (not a new AC):**
> Query order table → status is "shipped"

**Test Implementation:**
```javascript
test('AC-12 part 2: Order status persists in database', async () => {
  const book = await createBook('Test Book', 'Test Author', 10.00, 5);
  const orderId = createOrder(book.id, 2, 10.00);

  await request(app)
    .post(`/api/orders/${orderId}/ship`)
    .set('x-role', 'admin');

  const order = db.prepare('SELECT status FROM "order" WHERE id = ?').get(orderId);

  expect(order.status).toBe('shipped');
});
```

**Assertions:**
1. Direct database query confirms order status is "shipped"

**Traceability:**
- Maps to breakdown verification step 10: "Order status persists in database"
- Validates REQ-8 (order shipping updates status)

---

#### Test 3: AC-13 — Shopper ship order returns 403

**Acceptance Criterion:**
> **Given** an order with status "placed"  
> **When** a shopper POST `/api/orders/:id/ship` with `x-role: shopper`  
> **Then** the response is 403 with `{ "error": "<message>" }`

**Test Implementation:**
```javascript
test('AC-13: Shopper ship order returns 403', async () => {
  const book = await createBook('Test Book', 'Test Author', 10.00, 5);
  const orderId = createOrder(book.id, 2, 10.00);

  const response = await request(app)
    .post(`/api/orders/${orderId}/ship`)
    .set('x-role', 'shopper');

  expect(response.status).toBe(403);
  expect(response.body).toEqual({ error: 'Admin role required' });
});
```

**Assertions:**
1. HTTP status code is 403
2. Response body is exactly `{ "error": "Admin role required" }`

**Traceability:**
- Maps to AC-13 directly
- Maps to breakdown verification step 11: "Shopper 403 on ship"

---

#### Test 4: Ship nonexistent order returns 404

**Verification Step (not a new AC):**
> POST `/api/orders/999/ship` with admin → 404 "Order not found"

**Test Implementation:**
```javascript
test('Ship nonexistent order returns 404', async () => {
  const response = await request(app)
    .post('/api/orders/999/ship')
    .set('x-role', 'admin');

  expect(response.status).toBe(404);
  expect(response.body).toEqual({ error: 'Order not found' });
});
```

**Assertions:**
1. HTTP status code is 404
2. Response body is exactly `{ "error": "Order not found" }`

**Traceability:**
- Maps to breakdown verification step 12: "Ship nonexistent order → 404"
- Validates REQ-10 error handling (not-found case)

---

#### Test 5: Multi-item checkout creates order with multiple order_item rows

**Verification Step (not a new AC):**
> Create cart with 2 books, checkout → order has 2 order_item rows, both stocks reduced

**Test Implementation:**
```javascript
test('Multi-item checkout creates order with multiple order_item rows', async () => {
  const bookA = await createBook('Book A', 'Author 1', 10.00, 5);
  const bookB = await createBook('Book B', 'Author 2', 15.00, 3);

  const cartData = await request(app)
    .post('/api/cart/items')
    .send({ bookId: bookA.id, quantity: 2 });
  
  const cartId = cartData.body.cartId;

  await request(app)
    .post('/api/cart/items')
    .set('x-cart-id', cartId.toString())
    .send({ bookId: bookB.id, quantity: 1 });

  const response = await request(app)
    .post('/api/checkout')
    .set('x-cart-id', cartId.toString());

  expect(response.status).toBe(201);
  expect(response.body.items).toHaveLength(2);
  expect(response.body.total).toBe(35.00); // 2*10 + 1*15

  const orderId = response.body.id;
  const orderItemCount = db.prepare('SELECT COUNT(*) as count FROM order_item WHERE order_id = ?').get(orderId).count;
  expect(orderItemCount).toBe(2);

  const bookAResponse = await request(app).get(`/api/books/${bookA.id}`);
  expect(bookAResponse.body.stock).toBe(3); // 5 - 2

  const bookBResponse = await request(app).get(`/api/books/${bookB.id}`);
  expect(bookBResponse.body.stock).toBe(2); // 3 - 1
});
```

**Assertions:**
1. Checkout returns 201
2. Response body items array length is 2
3. Response body total is 35.00 (2×10 + 1×15)
4. Direct database query confirms 2 order_item rows
5. Book A stock reduced from 5 to 3
6. Book B stock reduced from 3 to 2

**Traceability:**
- Maps to breakdown verification step 13: "Multi-item checkout"
- Validates AC-10 behavior with multiple books

---

#### Test 6: Persistence across restart (order and stock)

**Acceptance Criterion (partial):**
> **Given** the server has created books, carts, and orders  
> **When** the server restarts  
> **Then** the order and reduced stock values are still present in the SQLite file

**Note:** This test provides **partial coverage** for AC-14. Full coverage is in Story 5's persistence tests.

**Test Implementation:**
```javascript
test('AC-14: Persistence across restart (order and stock)', async () => {
  const book = await createBook('Persist Test', 'Author', 10.00, 10);
  const cartData = await request(app)
    .post('/api/cart/items')
    .send({ bookId: book.id, quantity: 3 });
  
  const cartId = cartData.body.cartId;

  const checkoutResponse = await request(app)
    .post('/api/checkout')
    .set('x-cart-id', cartId.toString());

  const orderId = checkoutResponse.body.id;

  // Simulate restart by closing and reopening database
  db.close();
  const Database = require('better-sqlite3');
  const path = require('path');
  const dbPath = path.join(__dirname, '..', 'data', 'booknest.sqlite');
  const reopenedDb = new Database(dbPath);

  const orderCount = reopenedDb.prepare('SELECT COUNT(*) as count FROM "order"').get().count;
  expect(orderCount).toBe(1);

  const persistedStock = reopenedDb.prepare('SELECT stock FROM product WHERE id = ?').get(book.id).stock;
  expect(persistedStock).toBe(7); // 10 - 3

  const persistedStatus = reopenedDb.prepare('SELECT status FROM "order" WHERE id = ?').get(orderId).status;
  expect(persistedStatus).toBe('placed');

  reopenedDb.close();
});
```

**Assertions:**
1. After database close/reopen, order row exists
2. Stock reduction persists (10 → 7)
3. Order status persists ("placed")

**Traceability:**
- Maps to breakdown verification step 14: "Persistence across restart"
- Provides initial coverage for AC-14 (full coverage in Story 5)

---

## Complete Test File: __tests__/orders.test.js

```javascript
const request = require('supertest');
const app = require('../
