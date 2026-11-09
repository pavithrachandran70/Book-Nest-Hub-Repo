# Test Automation Summary — Story 03-Book

**Story:** Cart operations with stock validation  
**Repo:** Book  
**Story ID:** 03-Book  
**Generated:** 2025-06-10

---

## Overview

This document provides the automated test specification for Story 03-Book, which implements the cart API endpoints: adding items to a cart (creating the cart if needed), retrieving cart contents with calculated line totals, and removing items. Stock validation prevents adding quantities that exceed available inventory. All test cases map 1:1 to acceptance criteria and verification steps defined in the breakdown.

---

## Test Coverage Matrix

| AC/Verification Step | Test File | Test Case | Status |
|---------------------|-----------|-----------|--------|
| AC-6 part 1: Add book to new cart returns 201 with cartId | `__tests__/cart.test.js` | Test 1 | ✓ Implemented |
| AC-6 part 2: GET /api/cart shows line with price and lineTotal | `__tests__/cart.test.js` | Test 2 | ✓ Implemented |
| AC-7: Add same book again increments quantity | `__tests__/cart.test.js` | Test 3 | ✓ Implemented |
| AC-8 part 1: Add quantity > stock returns 400 | `__tests__/cart.test.js` | Test 4 | ✓ Implemented |
| AC-8 part 2: Cart unchanged after validation failure | `__tests__/cart.test.js` | Test 5 | ✓ Implemented |
| Verification: Add quantity > stock in fresh cart, no cart created | `__tests__/cart.test.js` | Test 6 | ✓ Implemented |
| AC-9: DELETE /api/cart/items/:bookId removes line | `__tests__/cart.test.js` | Test 7 | ✓ Implemented |
| Verification: POST without x-cart-id creates new cart | `__tests__/cart.test.js` | Test 8 | ✓ Implemented |
| Verification: GET with no x-cart-id returns 400 | `__tests__/cart.test.js` | Test 9 | ✓ Implemented |
| Verification: GET with nonexistent cartId returns 404 | `__tests__/cart.test.js` | Test 10 | ✓ Implemented |
| Verification: DELETE with nonexistent bookId returns 404 | `__tests__/cart.test.js` | Test 11 | ✓ Implemented |
| Verification: POST with invalid bookId returns 400 | `__tests__/cart.test.js` | Test 12 | ✓ Implemented |
| Verification: POST with invalid quantity returns 400 | `__tests__/cart.test.js` | Test 13 | ✓ Implemented |
| Verification: POST with nonexistent bookId returns 404 | `__tests__/cart.test.js` | Test 14 | ✓ Implemented |

**Total Test Count:** 14 tests (matches breakdown verification approach count exactly)

---

## Tests

### __tests__/cart.test.js

**Purpose:** Test suite for cart operations and stock validation.

**Test Framework:** Jest + Supertest

**Test Isolation:**
- `beforeEach`: Delete all rows from `product`, `cart`, and `cart_item` tables to ensure clean state
- `afterAll`: Close database connection

**Helper Functions:**
- `createBook(title, author, price, stock)`: Admin POST to create a book, returns the book object

**Test Cases:**

#### Test 1: AC-6 part 1 — Add book to new cart returns 201 with cartId

**Acceptance Criterion:**
> **Given** a book exists with stock 5  
> **When** a shopper POST `/api/cart/items` with `{ "bookId": <id>, "quantity": 2 }` and no `x-cart-id` header  
> **Then** the response is 201 with a body containing the cart ID

**Test Implementation:**
```javascript
test('AC-6 part 1: Add book to new cart returns 201 with cartId', async () => {
  const book = await createBook('Test Book', 'Test Author', 12.99, 5);

  const response = await request(app)
    .post('/api/cart/items')
    .send({ bookId: book.id, quantity: 2 });

  expect(response.status).toBe(201);
  expect(response.body).toMatchObject({
    cartId: expect.any(Number),
    bookId: book.id,
    quantity: 2
  });
});
```

**Assertions:**
1. HTTP status code is 201
2. Response body includes auto-generated `cartId` (integer)
3. Response body includes the `bookId` that was added
4. Response body shows `quantity: 2`

**Traceability:**
- Maps to AC-6 part 1 directly
- Maps to breakdown verification step 1: "POST /api/cart/items with no x-cart-id → 201 with cart id"

---

#### Test 2: AC-6 part 2 — GET /api/cart shows line with price and lineTotal

**Acceptance Criterion:**
> **And** GET `/api/cart` with that `x-cart-id` header returns the cart with one line showing the book, quantity 2, unit price, and line total

**Test Implementation:**
```javascript
test('AC-6 part 2: GET /api/cart shows line with price and lineTotal', async () => {
  const book = await createBook('Test Book', 'Test Author', 12.99, 5);

  const addResponse = await request(app)
    .post('/api/cart/items')
    .send({ bookId: book.id, quantity: 2 });

  const cartId = addResponse.body.cartId;

  const getResponse = await request(app)
    .get('/api/cart')
    .set('x-cart-id', cartId.toString());

  expect(getResponse.status).toBe(200);
  expect(getResponse.body.cartId).toBe(cartId);
  expect(getResponse.body.items).toHaveLength(1);
  expect(getResponse.body.items[0]).toMatchObject({
    bookId: book.id,
    title: 'Test Book',
    author: 'Test Author',
    quantity: 2,
    price: 12.99,
    lineTotal: 25.98
  });
});
```

**Assertions:**
1. HTTP status code is 200
2. Response body includes the correct `cartId`
3. Items array has exactly 1 entry
4. First item includes `bookId`, `title`, `author`, `quantity: 2`, `price: 12.99`, and `lineTotal: 25.98` (quantity × price)

**Traceability:**
- Maps to AC-6 part 2 directly
- Maps to breakdown verification step 2: "GET /api/cart shows line, price, and line total"

---

#### Test 3: AC-7 — Add same book again increments quantity

**Acceptance Criterion:**
> **Given** a cart already contains a book with quantity 2  
> **When** the shopper POST `/api/cart/items` with `{ "bookId": <same-id>, "quantity": 1 }` using the same `x-cart-id`  
> **Then** the line quantity becomes 3  
> **And** GET `/api/cart` reflects the updated quantity and line total

**Test Implementation:**
```javascript
test('AC-7: Add same book again increments quantity', async () => {
  const book = await createBook('Test Book', 'Test Author', 10.00, 5);

  const addResponse1 = await request(app)
    .post('/api/cart/items')
    .send({ bookId: book.id, quantity: 2 });

  const cartId = addResponse1.body.cartId;

  const addResponse2 = await request(app)
    .post('/api/cart/items')
    .set('x-cart-id', cartId.toString())
    .send({ bookId: book.id, quantity: 1 });

  expect(addResponse2.status).toBe(201);
  expect(addResponse2.body.quantity).toBe(3);

  const getResponse = await request(app)
    .get('/api/cart')
    .set('x-cart-id', cartId.toString());

  expect(getResponse.body.items).toHaveLength(1);
  expect(getResponse.body.items[0].quantity).toBe(3);
  expect(getResponse.body.items[0].lineTotal).toBe(30.00);
});
```

**Assertions:**
1. Second add returns 201 status
2. Second add response shows `quantity: 3` (2 + 1)
3. Cart still has only 1 item (same book, not duplicated)
4. Item quantity is 3
5. Line total is recalculated (3 × 10.00 = 30.00)

**Traceability:**
- Maps to AC-7 directly
- Maps to breakdown verification step 3: "Add same book again → quantity becomes 3"

---

#### Test 4: AC-8 part 1 — Add quantity > stock returns 400

**Acceptance Criterion:**
> **Given** a book has stock 5  
> **When** a shopper POST `/api/cart/items` with `{ "bookId": <id>, "quantity": 6 }`  
> **Then** the response is 400 with `{ "error": "<message>" }`

**Test Implementation:**
```javascript
test('AC-8 part 1: Add quantity > stock returns 400', async () => {
  const book = await createBook('Test Book', 'Test Author', 10.00, 5);

  const addResponse1 = await request(app)
    .post('/api/cart/items')
    .send({ bookId: book.id, quantity: 2 });

  const cartId = addResponse1.body.cartId;

  const addResponse2 = await request(app)
    .post('/api/cart/items')
    .set('x-cart-id', cartId.toString())
    .send({ bookId: book.id, quantity: 4 });

  expect(addResponse2.status).toBe(400);
  expect(addResponse2.body).toEqual({ error: 'Requested quantity exceeds available stock' });
});
```

**Assertions:**
1. HTTP status code is 400
2. Response body is exactly `{ "error": "Requested quantity exceeds available stock" }`
3. Cart already has quantity 2; adding 4 more would total 6, which exceeds stock of 5

**Traceability:**
- Maps to AC-8 part 1 directly
- Maps to breakdown verification step 4: "Add quantity > stock → 400"

---

#### Test 5: AC-8 part 2 — Cart unchanged after validation failure

**Acceptance Criterion:**
> **And** the cart is unchanged

**Test Implementation:**
```javascript
test('AC-8 part 2: Cart unchanged after validation failure', async () => {
  const book = await createBook('Test Book', 'Test Author', 10.00, 5);

  const addResponse1 = await request(app)
    .post('/api/cart/items')
    .send({ bookId: book.id, quantity: 2 });

  const cartId = addResponse1.body.cartId;

  await request(app)
    .post('/api/cart/items')
    .set('x-cart-id', cartId.toString())
    .send({ bookId: book.id, quantity: 4 });

  const getResponse = await request(app)
    .get('/api/cart')
    .set('x-cart-id', cartId.toString());

  expect(getResponse.body.items).toHaveLength(1);
  expect(getResponse.body.items[0].quantity).toBe(2);
});
```

**Assertions:**
1. After failed add, GET /api/cart returns 200
2. Items array still has exactly 1 entry
3. Quantity is still 2 (unchanged from before the failed add)

**Traceability:**
- Maps to AC-8 part 2 directly
- Maps to breakdown verification step 5: "Cart unchanged after validation failure"

---

#### Test 6: Add quantity > stock in fresh cart returns 400, no cart created

**Verification Step (not a new AC):**
> POST `/api/cart/items` with no `x-cart-id`, `quantity: 10` (> stock 5) → 400, no cart created

**Test Implementation:**
```javascript
test('Add quantity > stock in fresh cart returns 400, no cart created', async () => {
  const book = await createBook('Test Book', 'Test Author', 10.00, 5);

  const response = await request(app)
    .post('/api/cart/items')
    .send({ bookId: book.id, quantity: 10 });

  expect(response.status).toBe(400);
  expect(response.body).toEqual({ error: 'Requested quantity exceeds available stock' });

  const cartCount = db.prepare('SELECT COUNT(*) as count FROM cart').get().count;
  expect(cartCount).toBe(0);
});
```

**Assertions:**
1. HTTP status code is 400
2. Response body is `{ "error": "Requested quantity exceeds available stock" }`
3. Direct database query confirms no cart row was created (count is 0)

**Traceability:**
- Maps to breakdown verification step 6: "Add quantity > stock in fresh cart → 400, no cart created"
- Clarifies AC-8 edge case: validation should prevent cart creation if stock is exceeded

---

#### Test 7: AC-9 — DELETE /api/cart/items/:bookId removes line

**Acceptance Criterion:**
> **Given** a cart contains a line for a book  
> **When** the shopper DELETE `/api/cart/items/:bookId` with the cart's `x-cart-id`  
> **Then** the response is 200 or 204  
> **And** GET `/api/cart` no longer shows that line

**Test Implementation:**
```javascript
test('AC-9: DELETE /api/cart/items/:bookId removes line', async () => {
  const book = await createBook('Test Book', 'Test Author', 10.00, 5);

  const addResponse = await request(app)
    .post('/api/cart/items')
    .send({ bookId: book.id, quantity: 2 });

  const cartId = addResponse.body.cartId;

  const deleteResponse = await request(app)
    .delete(`/api/cart/items/${book.id}`)
    .set('x-cart-id', cartId.toString());

  expect(deleteResponse.status).toBe(200);
  expect(deleteResponse.body).toEqual({ message: 'Item removed' });

  const getResponse = await request(app)
    .get('/api/cart')
    .set('x-cart-id', cartId.toString());

  expect(getResponse.body.items).toHaveLength(0);
});
```

**Assertions:**
1. DELETE returns 200 status
2. DELETE response body is `{ "message": "Item removed" }`
3. After delete, GET /api/cart returns items array with length 0 (empty)

**Traceability:**
- Maps to AC-9 directly
- Maps to breakdown verification step 7: "DELETE removes line, cart becomes empty"

---

#### Test 8: POST without x-cart-id creates new cart

**Verification Step (not a new AC):**
> POST `/api/cart/items` twice without `x-cart-id` → creates two different carts

**Test Implementation:**
```javascript
test('POST without x-cart-id creates new cart', async () => {
  const book = await createBook('Test Book', 'Test Author', 10.00, 5);

  const response1 = await request(app)
    .post('/api/cart/items')
    .send({ bookId: book.id, quantity: 1 });

  const cartId1 = response1.body.cartId;

  const response2 = await request(app)
    .post('/api/cart/items')
    .send({ bookId: book.id, quantity: 1 });

  const cartId2 = response2.body.cartId;

  expect(cartId1).not.toBe(cartId2);
});
```

**Assertions:**
1. First POST returns a cart ID (cartId1)
2. Second POST (also without x-cart-id) returns a different cart ID (cartId2)
3. cartId1 ≠ cartId2 (two separate carts created)

**Traceability:**
- Maps to breakdown verification step 8: "POST without x-cart-id creates new cart"
- Clarifies AC-6 behavior: missing x-cart-id always creates a new cart

---

#### Test 9: GET /api/cart with no x-cart-id returns 400

**Verification Step (not a new AC):**
> GET `/api/cart` with no `x-cart-id` → 400 "Cart ID required"

**Test Implementation:**
```javascript
test('GET /api/cart with no x-cart-id returns 400', async () => {
  const response = await request(app).get('/api/cart');

  expect(response.status).toBe(400);
  expect(response.body).toEqual({ error: 'Cart ID required' });
});
```

**Assertions:**
1. HTTP status code is 400
2. Response body is exactly `{ "error": "Cart ID required" }`

**Traceability:**
- Maps to breakdown verification step 9: "GET with no x-cart-id → 400"
- Validates REQ-10 error handling (missing required header)

---

#### Test 10: GET /api/cart with nonexistent cartId returns 404

**Verification Step (not a new AC):**
> GET `/api/cart` with `x-cart-id: 999` (nonexistent) → 404 "Cart not found"

**Test Implementation:**
```javascript
test('GET /api/cart with nonexistent cartId returns 404', async () => {
  const response = await request(app)
    .get('/api/cart')
    .set('x-cart-id', '999');

  expect(response.status).toBe(404);
  expect(response.body).toEqual({ error: 'Cart not found' });
});
```

**Assertions:**
1. HTTP status code is 404
2. Response body is exactly `{ "error": "Cart not found" }`

**Traceability:**
- Maps to breakdown verification step 10: "GET with nonexistent cartId → 404"
- Validates REQ-10 error handling (not-found case)

---

#### Test 11: DELETE with nonexistent bookId returns 404

**Verification Step (not a new AC):**
> DELETE `/api/cart/items/999` with valid `x-cart-id` → 404 "Item not found in cart"

**Test Implementation:**
```javascript
test('DELETE with nonexistent bookId returns 404', async () => {
  const book = await createBook('Test Book', 'Test Author', 10.00, 5);

  const addResponse = await request(app)
    .post('/api/cart/items')
    .send({ bookId: book.id, quantity: 1 });

  const cartId = addResponse.body.cartId;

  const response = await request(app)
    .delete('/api/cart/items/999')
    .set('x-cart-id', cartId.toString());

  expect(response.status).toBe(404);
  expect(response.body).toEqual({ error: 'Item not found in cart' });
});
```

**Assertions:**
1. HTTP status code is 404
2. Response body is exactly `{ "error": "Item not found in cart" }`
3. Cart exists and has one item, but book ID 999 is not in the cart

**Traceability:**
- Maps to breakdown verification step 11: "DELETE with nonexistent bookId → 404"
- Validates REQ-10 error handling (not-found case for cart item)

---

#### Test 12: POST with invalid bookId returns 400

**Verification Step (not a new AC):**
> POST `/api/cart/items` with `bookId: -1` → 400 "Invalid book ID"

**Test Implementation:**
```javascript
test('POST with invalid bookId returns 400', async () => {
  const response = await request(app)
    .post('/api/cart/items')
    .send({ bookId: -1, quantity: 1 });

  expect(response.status).toBe(400);
  expect(response.body).toEqual({ error: 'Invalid book ID' });
});
```

**Assertions:**
1. HTTP status code is 400
2. Response body is exactly `{ "error": "Invalid book ID" }`

**Traceability:**
- Maps to breakdown verification step 12: "POST with invalid bookId → 400"
- Validates REQ-6 validation rules (bookId must be positive integer)

---

#### Test 13: POST with invalid quantity returns 400

**Verification Step (not a new AC):**
> POST `/api/cart/items` with `quantity: 0` → 400 "Quantity must be a positive integer"

**Test Implementation:**
```javascript
test('POST with invalid quantity returns 400', async () => {
  const book = await createBook('Test Book', 'Test Author', 10.00, 5);

  const response = await request(app)
    .post('/api/cart/items')
    .send({ bookId: book.id, quantity: 0 });

  expect(response.status).toBe(400);
  expect(response.body).toEqual({ error: 'Quantity must be a positive integer' });
});
```

**Assertions:**
1. HTTP status code is 400
2. Response body is exactly `{ "error": "Quantity must be a positive integer" }`

**Traceability:**
- Maps to breakdown verification step 13: "POST with invalid quantity → 400"
- Validates REQ-6 validation rules (quantity must be positive)

---

#### Test 14: POST with nonexistent bookId returns 404

**Verification Step (not a new AC):**
> POST `/api/cart/items` with `bookId: 999` (does not exist) → 404 "Book not found"

**Test Implementation:**
```javascript
test('POST with nonexistent bookId returns 404', async () => {
  const response = await request(app)
    .post('/api/cart/items')
    .send({ bookId: 999, quantity: 1 });

  expect(response.status).toBe(404);
  expect(response.body).toEqual({ error: 'Book not found' });
});
```

**Assertions:**
1. HTTP status code is 404
2. Response body is exactly `{ "error": "Book not found" }`

**Traceability:**
- Maps to breakdown verification step 14: "POST with nonexistent bookId → 404"
- Validates REQ-6 validation rules (bookId must exist in product table)

---

## Complete Test File

**File:** `__tests__/cart.test.js`

```javascript
const request = require('supertest');
const app = require('../src/app');
const { db } = require('../src/db');

describe('Cart operations and stock validation', () => {
  // Helper to create a book (admin role)
  async function createBook(title, author, price, stock) {
    const response = await request(app)
      .post('/api/books')
      .set('x-role', 'admin')
      .send({ title, author, price, stock });
    return response.body;
  }

  beforeEach(() => {
    db.prepare('DELETE FROM cart_item').run();
    db.prepare('DELETE FROM cart').run();
    db.prepare('DELETE FROM product').run();
  });

  afterAll(() => {
    db.close();
  });

  test('AC-6 part 1: Add book to new cart returns 201 with cartId', async () => {
    const book = await createBook('Test Book', 'Test Author', 12.99, 5);

    const response = await request(app)
      .post('/api/cart/items')
      .send({ bookId: book.id, quantity: 2 });

    expect(response.status).toBe(201);
    expect(response.body).toMatchObject({
      cartId: expect.any(Number),
      bookId: book.id,
      quantity: 2
    });
  });

  test('AC-6 part 2: GET /api/cart shows line with price and lineTotal', async () => {
    const book = await createBook('Test Book', 'Test Author', 12.99, 5);

    const addResponse = await request(app)
      .post('/api/cart/items')
      .send({ bookId: book.id, quantity: 2 });

    const cartId = addResponse.body.cartId;

    const getResponse = await request(app)
      .get('/api/cart')
      .set('x-cart-id', cartId.toString());

    expect(getResponse.status).toBe(200);
    expect(getResponse.body.cartId).toBe(cartId);
    expect(getResponse.body.items).toHaveLength(1);
    expect(getResponse.body.items[0]).toMatchObject({
      bookId: book.id,
      title: 'Test Book',
      author: 'Test Author',
      quantity: 2,
      price: 12.99,
      lineTotal: 25.98
    });
  });

  test('AC-7: Add same book again increments quantity', async () => {
    const book = await createBook('Test Book', 'Test Author', 10.00, 5);

    const addResponse1 = await request(app)
      .post('/api/cart/items')
      .send({ bookId: book.id, quantity: 2 });

    const cartId = addResponse1.body.cartId;

    const addResponse2 = await request(app)
      .post('/api/cart/items')
      .set('x-cart-id', cartId.toString())
      .send({ bookId: book.id, quantity: 1 });

    expect(addResponse2.status).toBe(201);
    expect(addResponse2.body.quantity).toBe(3);

    const getResponse = await request(app)
      .get('/api/cart')
      .set('x-cart-id', cartId.toString());

    expect(getResponse.body.items).toHaveLength(1);
    expect(getResponse.body.items[0].quantity).toBe(3);
    expect(getResponse.body.items[0].lineTotal).toBe(30.00);
  });

  test('AC-8 part 1: Add quantity > stock returns 400', async () => {
    const book = await createBook('Test Book', 'Test Author', 10.00, 5);

    const addResponse1 = await request(app)
      .post('/api/cart/items')
      .send({ bookId: book.id, quantity: 2 });

    const cartId = addResponse1.body.cartId;

    const addResponse2 = await request(app)
      .post('/api/cart/items')
      .set('x-cart-id', cartId.toString())
      .send({ bookId: book.id, quantity: 4 });

    expect(addResponse2.status).toBe(400);
    expect(addResponse2.body).toEqual({ error: 'Requested quantity exceeds available stock' });
  });

  test('AC-8 part 2: Cart unchanged after validation failure', async () => {
    const book = await createBook('Test Book', 'Test Author', 10.00, 5);

    const addResponse1 = await request(app)
      .post('/api/cart/items')
      .send({ bookId: book.id, quantity: 2 });

    const cartId = addResponse1.body.cartId;

    await request(app)
      .post('/api/cart/items')
      .set('x-cart-id', cartId.toString())
      .send({ bookId: book.id, quantity: 4 });

    const getResponse = await request(app)
      .get('/api/cart')
      .set('x-cart-id', cartId.toString());

    expect(getResponse.body.items).toHaveLength(1);
    expect(getResponse.body.items[0].quantity).toBe(2);
  });

  test('Add quantity > stock in fresh cart returns 400, no cart created', async () => {
    const book = await createBook('Test Book', 'Test Author', 10.00, 5);

    const response = await request(app)
      .post('/api/cart/items')
      .send({ bookId: book.id, quantity: 10 });

    expect(response.status).toBe(400);
    expect(response.body).toEqual({ error: 'Requested quantity exceeds available stock' });

    const cartCount = db.prepare('SELECT COUNT(*) as count FROM cart').get().count;
    expect(cartCount).toBe(0);
  });

  test('AC-9: DELETE /api/cart/items/:bookId removes line', async () => {
    const book = await createBook('Test Book', 'Test Author', 10.00, 5);

    const addResponse = await request(app)
      .post('/api/cart/items')
      .send({ bookId: book.id, quantity: 2 });

    const cartId = addResponse.body.cartId;

    const deleteResponse = await request(app)
      .delete(`/api/cart/items/${book.id}`)
      .set('x-cart-id', cartId.toString());

    expect(deleteResponse.status).toBe(200);
    expect(deleteResponse.body).toEqual({ message: 'Item removed' });

    const getResponse = await request(app)
      .get('/api/cart')
      .set('x-cart-id', cartId.toString());

    expect(getResponse.body.items).toHaveLength(0);
  });

  test('POST without x-cart-id creates new cart', async () => {
    const book = await createBook('Test Book', 'Test Author', 10.00, 5);

    const response1 = await request(app)
      .post('/api/cart/items')
      .send({ bookId: book.id, quantity: 1 });

    const cartId1 = response1.body.cartId;

    const response2 = await request(app)
      .post('/api/cart/items')
      .send({ bookId: book.id, quantity: 1 });

    const cartId2 = response2.body.cartId;

    expect(cartId1).not.toBe(cartId2);
  });

  test('GET /api/cart with no x-cart-id returns 400', async () => {
    const response = await request(app).get('/api/cart');

    expect(response.status).toBe(400);
    expect(response.body).toEqual({ error: 'Cart ID required' });
  });

  test('GET /api/cart with nonexistent cartId returns 404', async () => {
    const response = await request(app)
      .get('/api/cart')
      .set('x-cart-id', '999');

    expect(response.status).toBe(404);
    expect(response.body).toEqual({ error: 'Cart not found' });
  });

  test('DELETE with nonexistent bookId returns 404', async () => {
    const book = await createBook('Test Book', 'Test Author', 10.00, 5);

    const addResponse = await request(app)
      .post('/api/cart/items')
      .send({ bookId: book.id, quantity: 1 });

    const cartId = addResponse.body.cartId;

    const response = await request(app)
      .delete('/api/cart/items/999')
      .set('x-cart-id', cartId.toString());

    expect(response.status).toBe(404);
    expect(response.body).toEqual({ error: 'Item not found in cart' });
  });

  test('POST with invalid bookId returns 400', async () => {
    const response = await request(app)
      .post('/api/cart/items')
      .send({ bookId: -1, quantity: 1 });

    expect(response.status).toBe(400);
    expect(response.body).toEqual({ error: 'Invalid book ID' });
  });

  test('POST with invalid quantity returns 400', async () => {
    const book = await createBook('Test Book
