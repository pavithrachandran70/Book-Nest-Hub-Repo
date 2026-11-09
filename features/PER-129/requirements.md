# Requirements — PER-129

**Derived from:** story-per-129.md  
**Last updated:** 2025-06-10

---

## Overview

Build a complete Node.js bookstore API with SQLite that allows shoppers to browse books, manage a cart, and place orders, while admins can add books and mark orders shipped. The system must support full end-to-end flows from catalog management through checkout and shipping, with role-based access control.

---

## Acceptance Criteria

### AC-1: Project scaffold and database initialization
**Given** a fresh clone of the repository  
**When** I run `npm install` and `npm start`  
**Then** the API listens on port 3000 and creates `data/booknest.sqlite` with the following tables:
- `product` (id, title, author, price, stock)
- `cart` (id, created_at)
- `cart_item` (cart_id, book_id, quantity)
- `order` (id, status, total, created_at)
- `order_item` (order_id, book_id, quantity, price)

### AC-2: Health check endpoint
**Given** the server is running  
**When** I GET `/api/health`  
**Then** the response is 200 with body `{ "status": "ok" }`

### AC-3: Admin can add books to catalog
**Given** an admin request with `x-role: admin` header  
**When** they POST `/api/books` with `{ "title": "<title>", "author": "<author>", "price": <number>, "stock": <number> }`  
**Then** the response is 201 with the created book object  
**And** GET `/api/books/:id` returns that book with all fields

### AC-4: Shopper cannot add books
**Given** a shopper request with `x-role: shopper` header  
**When** they POST `/api/books`  
**Then** the response is 403 with `{ "error": "<message>" }`

### AC-5: List books in catalog
**Given** books exist in the catalog  
**When** a shopper GET `/api/books`  
**Then** the response is 200 with an array of books, each containing `id`, `title`, `author`, `price`, and `stock`

### AC-6: Add item to new cart
**Given** a book exists with stock 5  
**When** a shopper POST `/api/cart/items` with `{ "bookId": <id>, "quantity": 2 }` and no `x-cart-id` header  
**Then** the response is 201 with a body containing the cart ID  
**And** GET `/api/cart` with that `x-cart-id` header returns the cart with one line showing the book, quantity 2, unit price, and line total

### AC-7: Increment quantity for existing cart line
**Given** a cart already contains a book with quantity 2  
**When** the shopper POST `/api/cart/items` with `{ "bookId": <same-id>, "quantity": 1 }` using the same `x-cart-id`  
**Then** the line quantity becomes 3  
**And** GET `/api/cart` reflects the updated quantity and line total

### AC-8: Reject quantity above stock
**Given** a book has stock 5  
**When** a shopper POST `/api/cart/items` with `{ "bookId": <id>, "quantity": 6 }`  
**Then** the response is 400 with `{ "error": "<message>" }`  
**And** the cart is unchanged

### AC-9: Remove item from cart
**Given** a cart contains a line for a book  
**When** the shopper DELETE `/api/cart/items/:bookId` with the cart's `x-cart-id`  
**Then** the response is 200 or 204  
**And** GET `/api/cart` no longer shows that line

### AC-10: Checkout creates order and empties cart
**Given** a cart with quantities that are in stock  
**When** the shopper POST `/api/checkout` with that `x-cart-id`  
**Then** the response is 201 with an order object containing `id`, `status: "placed"`, the same line items, and `total`  
**And** GET `/api/cart` for that cart returns an empty cart  
**And** each book's stock is reduced by the ordered quantity

### AC-11: Checkout fails when quantity exceeds stock
**Given** a cart line quantity exceeds the book's current stock  
**When** the shopper POST `/api/checkout`  
**Then** the response is 400 with `{ "error": "<message>" }`  
**And** no order is created  
**And** stock is unchanged

### AC-12: Admin can ship orders
**Given** an order with status "placed"  
**When** an admin POST `/api/orders/:id/ship` with `x-role: admin`  
**Then** the response is 200 and the order status becomes "shipped"

### AC-13: Shopper cannot ship orders
**Given** an order with status "placed"  
**When** a shopper POST `/api/orders/:id/ship` with `x-role: shopper`  
**Then** the response is 403 with `{ "error": "<message>" }`

### AC-14: Data persists across restarts
**Given** the server has created books, carts, and orders  
**When** the server restarts  
**Then** GET `/api/books/:id`, GET `/api/orders/:id`, and the reduced stock values are all still present in the SQLite file

### AC-15: Test coverage
**Given** the repository contains the application  
**When** I run `npm test`  
**Then** all tests pass, covering:
- Health check endpoint
- Catalog operations with admin/shopper auth
- Cart operations (add, update, delete)
- Checkout with stock validation
- Shipping with role enforcement
- Data persistence across restarts

---

## Requirements

### REQ-1: Project structure and dependencies
- Create `package.json` with Express, better-sqlite3, and Jest as dependencies
- Create `src/server.js` as the entry point
- Create `data/` directory for SQLite database file
- Provide `npm start` script to launch the server
- Provide `npm test` script to run Jest tests

### REQ-2: Database schema
- **product** table: `id` (primary key), `title` (text), `author` (text), `price` (real), `stock` (integer)
- **cart** table: `id` (primary key), `created_at` (timestamp)
- **cart_item** table: `cart_id` (foreign key), `book_id` (foreign key), `quantity` (integer), composite primary key on (cart_id, book_id)
- **order** table: `id` (primary key), `status` (text), `total` (real), `created_at` (timestamp)
- **order_item** table: `order_id` (foreign key), `book_id` (integer), `quantity` (integer), `price` (real), composite primary key on (order_id, book_id)
- Create tables automatically on server startup if they do not exist

### REQ-3: Role-based access control
- Read `x-role` header from requests
- Accept `admin` and `shopper` as valid roles
- POST `/api/books` requires `x-role: admin`, return 403 otherwise
- POST `/api/orders/:id/ship` requires `x-role: admin`, return 403 otherwise
- All other endpoints allow any role or no role

### REQ-4: Catalog endpoints
- **POST /api/books**: Accept `{ "title", "author", "price", "stock" }`, insert into `product` table, return 201 with created book (admin only)
- **GET /api/books**: Return 200 with array of all books, each with `id`, `title`, `author`, `price`, `stock`
- **GET /api/books/:id**: Return 200 with single book object, or 404 if not found

### REQ-5: Cart identification and management
- Use `x-cart-id` header to identify the shopper's cart
- If `x-cart-id` is missing on POST `/api/cart/items`, create a new cart and return the cart ID in the response
- If `x-cart-id` is present, use that cart
- Store cart ID as an integer primary key

### REQ-6: Cart item operations
- **POST /api/cart/items**: Accept `{ "bookId", "quantity" }`
  - Validate that `bookId` exists and `quantity` is positive
  - Validate that `quantity` does not exceed `stock`, return 400 if it does
  - If the book is already in the cart, increment the quantity by the request amount
  - If the book is not in the cart, insert a new `cart_item` row
  - Return 201 with cart ID included
- **GET /api/cart**: Accept `x-cart-id` header
  - Return 200 with cart object containing an array of line items
  - Each line item includes `bookId`, `title`, `author`, `quantity`, `price`, and `lineTotal` (quantity × price)
- **DELETE /api/cart/items/:bookId**: Accept `x-cart-id` header
  - Remove the `cart_item` row for that cart and book
  - Return 200 or 204

### REQ-7: Checkout and order creation
- **POST /api/checkout**: Accept `x-cart-id` header
  - Validate that all cart item quantities are ≤ current stock, return 400 otherwise
  - Create an `order` row with status "placed" and calculated total
  - Create `order_item` rows copying `cart_item` data with current price
  - Reduce each book's `stock` by the ordered quantity
  - Delete all `cart_item` rows for that cart
  - Return 201 with order object containing `id`, `status`, `items` array, and `total`

### REQ-8: Order shipping
- **POST /api/orders/:id/ship**: Accept `x-role: admin`
  - Update the order's status to "shipped"
  - Return 200 with updated order object
  - Return 403 if role is not admin
  - Return 404 if order does not exist

### REQ-9: Health check
- **GET /api/health**: Return 200 with `{ "status": "ok" }`

### REQ-10: Error handling
- All validation failures return 400 with `{ "error": "<descriptive message>" }`
- All authorization failures return 403 with `{ "error": "<message>" }`
- All not-found cases return 404 with `{ "error": "<message>" }`
- All internal errors return 500 with `{ "error": "Internal server error" }`

### REQ-11: Data persistence
- Use `data/booknest.sqlite` as the database file path
- Ensure all operations use synchronous better-sqlite3 methods or transactions where needed
- Verify data survives server restart by testing reads after restart

### REQ-12: Test coverage
- Write Jest tests covering all acceptance criteria
- Include tests for role enforcement (admin vs shopper)
- Include tests for validation (stock limits, missing fields)
- Include tests for state transitions (cart → order, placed → shipped)
- Include tests for data persistence (restart scenario)

---

## Open Questions

None. The ticket provides complete acceptance criteria and scope boundaries.

---

## Notes

- The ticket explicitly excludes frontend, accounts, real payments, and multi-shop support.
- The `x-role` header is the only authentication mechanism; no password or OAuth is required.
- The `x-cart-id` header identifies a cart; no session or cookie mechanism is needed.
- All prices and totals are handled as numbers; no currency formatting is specified.
- The schema design must support the cart-to-order flow with stock reduction.
