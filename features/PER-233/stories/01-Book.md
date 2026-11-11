# Story — Add GET /api/orders/:id endpoint

**ID:** PER-233  
**Type:** story  
**Status:** planned  
**Epic:** (none)  
**Covers:** (none)

---

## Description

Add `GET /api/orders/:id` to the BookNest API so a placed order can be read back. The endpoint returns the order with all fields and items, or a 404 for unknown IDs. Both shopper and admin roles can call it.

---

## Given/When/Then Acceptance Criteria

### AC-1: Retrieve an existing order with all fields

**Given** an order exists in the database with id `123`, status `placed`, total `29.97`, createdAt timestamp, and three items  
**When** a request is made to `GET /api/orders/123` with header `x-role: shopper`  
**Then** the response status is `200`  
**And** the response body contains:
```json
{
  "id": 123,
  "status": "placed",
  "total": 29.97,
  "createdAt": "<ISO 8601 timestamp>",
  "items": [
    { "bookId": 1, "quantity": 2, "price": 9.99 },
    { "bookId": 2, "quantity": 1, "price": 9.99 }
  ]
}
```

---

### AC-2: Retrieve a shipped order

**Given** an order exists with id `456` and status `shipped`  
**When** a request is made to `GET /api/orders/456` with header `x-role: admin`  
**Then** the response status is `200`  
**And** the response body field `status` is `"shipped"`

---

### AC-3: Return 404 for unknown order id

**Given** no order exists with id `999`  
**When** a request is made to `GET /api/orders/999` with header `x-role: shopper`  
**Then** the response status is `404`  
**And** the response body is:
```json
{
  "error": "Order not found"
}
```

---

### AC-4: Admin role can retrieve an order

**Given** an order exists with id `789`  
**When** a request is made to `GET /api/orders/789` with header `x-role: admin`  
**Then** the response status is `200`  
**And** the order data is returned

---

### AC-5: Shopper role can retrieve an order

**Given** an order exists with id `789`  
**When** a request is made to `GET /api/orders/789` with header `x-role: shopper`  
**Then** the response status is `200`  
**And** the order data is returned

---

## File Plan

### Modified Files

**`src/routes/orders.js`**  
Add the `GET /:id` route handler to the existing orders router. If this file does not exist, create it with the Express router setup and this new route.

**`src/server.js`** (or `src/routes/index.js` if routes are centralized)  
Ensure the orders router is mounted at `/api/orders` if not already present:
```javascript
const ordersRouter = require('./routes/orders');
app.use('/api/orders', ordersRouter);
```

### New Files

**`src/routes/orders.test.js`**  
Jest test suite covering all five acceptance criteria plus edge cases.

---

## Interfaces

### Request

**HTTP Method:** `GET`  
**Path:** `/api/orders/:id`  
**Headers:**
- `x-role` (required): `"shopper"` or `"admin"`

**URL Parameters:**
- `:id` (integer): The order ID to retrieve

### Response

**Success (200):**
```json
{
  "id": 123,
  "status": "placed",
  "total": 29.97,
  "createdAt": "2025-01-15T10:30:00.000Z",
  "items": [
    { "bookId": 1, "quantity": 2, "price": 9.99 },
    { "bookId": 2, "quantity": 1, "price": 9.99 }
  ]
}
```

**Fields:**
- `id` (integer): The order ID
- `status` (string): `"placed"` or `"shipped"`
- `total` (number): The order total
- `createdAt` (string): ISO 8601 timestamp
- `items` (array): Array of item objects, each with:
  - `bookId` (integer)
  - `quantity` (integer)
  - `price` (number)

**Not Found (404):**
```json
{
  "error": "Order not found"
}
```

### Database Tables (existing, no changes)

**`orders`**  
Columns: `id`, `status`, `total`, `createdAt`

**`order_items`**  
Columns: `orderId`, `bookId`, `quantity`, `price`

---

## Algorithm Notes

### Route Handler Logic (`src/routes/orders.js`)

1. **Extract parameters and headers:**
   - Extract `:id` from `req.params.id`
   - Parse as integer
   - Extract `x-role` from `req.headers['x-role']`

2. **Validate role:**
   - Check that `x-role` is either `"shopper"` or `"admin"`
   - (Optional: If role validation is handled by existing middleware, skip this step)

3. **Query the database:**
   - Use better-sqlite3 to execute the following prepared statement:
     ```sql
     SELECT 
       o.id, 
       o.status, 
       o.total, 
       o.createdAt,
       oi.bookId,
       oi.quantity,
       oi.price
     FROM orders o
     LEFT JOIN order_items oi ON o.id = oi.orderId
     WHERE o.id = ?
     ```
   - Bind the order ID as the parameter

4. **Handle empty result:**
   - If query returns zero rows:
     - Respond with status `404`
     - Body: `{ "error": "Order not found" }`
     - Return early

5. **Map query results:**
   - First row contains `id`, `status`, `total`, `createdAt` (shared across all rows)
   - All rows contain `bookId`, `quantity`, `price` (one per item)
   - Aggregate:
     ```javascript
     const order = {
       id: rows[0].id,
       status: rows[0].status,
       total: rows[0].total,
       createdAt: rows[0].createdAt,
       items: rows.map(row => ({
         bookId: row.bookId,
         quantity: row.quantity,
         price: row.price
       }))
     };
     ```
   - Note: If the order has zero items, `LEFT JOIN` returns one row with null `bookId`, `quantity`, `price`. Filter these out:
     ```javascript
     items: rows
       .filter(row => row.bookId !== null)
       .map(row => ({ bookId: row.bookId, quantity: row.quantity, price: row.price }))
     ```

6. **Respond with success:**
   - Status `200`
   - Body: the `order` object

### Database Access Pattern

- Use the existing better-sqlite3 database connection (likely imported or passed as a dependency in `src/routes/orders.js`)
- Use `.prepare()` and `.all()` to execute the query and retrieve all matching rows
- Example:
  ```javascript
  const db = require('../db'); // or however the connection is accessed
  const stmt = db.prepare(`
    SELECT 
      o.id, 
      o.status, 
      o.total, 
      o.createdAt,
      oi.bookId,
      oi.quantity,
      oi.price
    FROM orders o
    LEFT JOIN order_items oi ON o.id = oi.orderId
    WHERE o.id = ?
  `);
  const rows = stmt.all(orderId);
  ```

### Express Router Registration

In `src/routes/orders.js`:
```javascript
const express = require('express');
const router = express.Router();

router.get('/:id', (req, res) => {
  // Implementation here
});

module.exports = router;
```

In `src/server.js` (or wherever the Express app is configured):
```javascript
const ordersRouter = require('./routes/orders');
app.use('/api/orders', ordersRouter);
```

---

## Tests

**File:** `src/routes/orders.test.js`

### Test Coverage

Total test cases: **8** (5 ACs + 3 edge cases from breakdown verification approach)

1. **AC-1: Retrieve an existing order with all fields**  
   - Seed order `123` with status `"placed"`, total `29.97`, three items
   - Call `GET /api/orders/123` with `x-role: shopper`
   - Assert status `200`
   - Assert response body matches expected structure exactly (id, status, total, createdAt, items array with 3 elements)

2. **AC-2: Retrieve a shipped order**  
   - Seed order `456` with status `"shipped"`
   - Call `GET /api/orders/456` with `x-role: admin`
   - Assert status `200`
   - Assert `status` field is `"shipped"`

3. **AC-3: Return 404 for unknown order id**  
   - Do not seed order `999`
   - Call `GET /api/orders/999` with `x-role: shopper`
   - Assert status `404`
   - Assert body is `{ "error": "Order not found" }`

4. **AC-4: Admin role can retrieve an order**  
   - Seed order `789`
   - Call `GET /api/orders/789` with `x-role: admin`
   - Assert status `200`
   - Assert order data is returned with correct structure

5. **AC-5: Shopper role can retrieve an order**  
   - Call `GET /api/orders/789` with `x-role: shopper` (same order as AC-4)
   - Assert status `200`
   - Assert order data is returned with correct structure

6. **Edge case: Order with zero items**  
   - Seed order with no corresponding `order_items` rows
   - Call `GET /api/orders/<id>` with valid role
   - Assert status `200`
   - Assert `items` array is empty `[]`

7. **Edge case: `createdAt` is a valid ISO 8601 string**  
   - Seed order with a known timestamp
   - Call `GET /api/orders/<id>`
   - Assert `createdAt` matches ISO 8601 format (e.g., regex or parsing with `Date`)

8. **Edge case: `total` and `price` fields are numbers, not strings**  
   - Seed order with numeric `total` and item `price` values
   - Call `GET /api/orders/<id>`
   - Assert `typeof response.total === 'number'`
   - Assert `typeof response.items[0].price === 'number'`

### Test Setup

- Use Jest with Supertest for HTTP assertions
- Use an in-memory SQLite database or a test fixture database file
- Seed data in `beforeEach` or per-test setup blocks
- Clean up database state in `afterEach`

---

## Notes

- **Port correction:** The ticket specifies port 3001, but the repo context states default port 3000 via `PORT` environment variable. The implementation should respect the `PORT` environment variable; tests should use the actual server port or mock the app directly.
- **Timestamp format:** The database `createdAt` column likely stores an integer (Unix timestamp) or a string. The response must convert this to an ISO 8601 string. Use `new Date(createdAt).toISOString()` if needed.
- **No authentication beyond `x-role`:** The repo context confirms `x-role` header is the only identity check. No additional middleware or token validation is required.
- **Existing endpoints:** Do not modify health, books, cart, checkout, or ship endpoints.
- **Error format:** Follow the existing pattern `{ "error": "<message>" }` for all error responses.
