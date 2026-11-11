# Requirements — PER-233

**Add `GET /api/orders/:id` so a placed order can be read back**

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

## Verification

1. Create test orders in the SQLite database with known id, status, total, createdAt, and items
2. Call `GET /api/orders/:id` for each AC scenario with appropriate `x-role` header
3. Assert response status codes match expected values (200 or 404)
4. Assert response body structure and field values match the specification
5. Verify both `shopper` and `admin` roles return 200 for valid orders
6. Verify unknown order ids return 404 with correct error message

---

## Notes

- Endpoint: `GET /api/orders/:id` on existing BookNest API (port 3001)
- No new database tables required
- Do not modify existing endpoints: health, books, cart, checkout, ship
- Response field `status` values: `placed` or `shipped`
- Each item in `items` array contains: `bookId`, `quantity`, `price`
