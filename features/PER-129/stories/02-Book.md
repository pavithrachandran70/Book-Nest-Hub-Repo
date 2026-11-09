# Test Automation Summary — Story 02-Book

**Story:** Catalog endpoints with role-based access control  
**Repo:** Book  
**Story ID:** 02-Book  
**Generated:** 2025-06-10

---

## Overview

This document provides the automated test specification for Story 02-Book, which implements role-based access control and catalog API endpoints (POST, GET list, GET by ID). All test cases map 1:1 to acceptance criteria and verification steps defined in the breakdown.

---

## Test Coverage Matrix

| AC/Verification Step | Test File | Test Case | Status |
|---------------------|-----------|-----------|--------|
| AC-3 part 1: Admin POST creates book | `__tests__/books.test.js` | Test 1 | ✓ Implemented |
| AC-3 part 2: GET /api/books/:id returns book | `__tests__/books.test.js` | Test 2 | ✓ Implemented |
| AC-4: Shopper POST returns 403 | `__tests__/books.test.js` | Test 3 | ✓ Implemented |
| Verification: No x-role header returns 403 | `__tests__/books.test.js` | Test 4 | ✓ Implemented |
| Verification: Empty title returns 400 | `__tests__/books.test.js` | Test 5 | ✓ Implemented |
| Verification: Price = 0 returns 400 | `__tests__/books.test.js` | Test 6 | ✓ Implemented |
| Verification: Negative stock returns 400 | `__tests__/books.test.js` | Test 7 | ✓ Implemented |
| AC-5: GET /api/books returns array | `__tests__/books.test.js` | Test 8 | ✓ Implemented |
| Verification: Empty catalog returns [] | `__tests__/books.test.js` | Test 9 | ✓ Implemented |
| Verification: Nonexistent book returns 404 | `__tests__/books.test.js` | Test 10 | ✓ Implemented |
| Verification: List all books ordered by ID | `__tests__/books.test.js` | Test 11 | ✓ Implemented |

**Total Test Count:** 11 tests (matches breakdown verification approach count exactly)

---

## Tests

### __tests__/books.test.js

**Purpose:** Test suite for catalog endpoints and role enforcement.

**Test Framework:** Jest + Supertest

**Test Isolation:**
- `beforeEach`: Delete all rows from `product` table to ensure clean state
- `afterAll`: Close database connection

**Test Cases:**

#### Test 1: AC-3 part 1 — Admin POST /api/books creates book and returns 201

**Acceptance Criterion:**
> **Given** an admin request with `x-role: admin` header  
> **When** they POST `/api/books` with `{ "title": "<title>", "author": "<author>", "price": <number>, "stock": <number> }`  
> **Then** the response is 201 with the created book object

**Test Implementation:**
```javascript
test('AC-3 part 1: Admin POST /api/books creates book and returns 201', async () => {
  const bookData = {
    title: 'The Pragmatic Programmer',
    author: 'Hunt & Thomas',
    price: 29.99,
    stock: 10
  };

  const response = await request(app)
    .post('/api/books')
    .set('x-role', 'admin')
    .send(bookData);

  expect(response.status).toBe(201);
  expect(response.body).toMatchObject({
    id: expect.any(Number),
    title: 'The Pragmatic Programmer',
    author: 'Hunt & Thomas',
    price: 29.99,
    stock: 10
  });
});
```

**Assertions:**
1. HTTP status code is 201
2. Response body includes auto-generated `id` (integer)
3. All request fields (title, author, price, stock) are returned unchanged

**Traceability:**
- Maps to AC-3 part 1 directly
- Maps to breakdown verification step 1: "Admin POST with valid data → 201"

---

#### Test 2: AC-3 part 2 — GET /api/books/:id returns created book

**Acceptance Criterion:**
> **And** GET `/api/books/:id` returns that book with all fields

**Test Implementation:**
```javascript
test('AC-3 part 2: GET /api/books/:id returns created book', async () => {
  const bookData = {
    title: 'Clean Code',
    author: 'Robert C. Martin',
    price: 34.99,
    stock: 5
  };

  const createResponse = await request(app)
    .post('/api/books')
    .set('x-role', 'admin')
    .send(bookData);

  const bookId = createResponse.body.id;

  const getResponse = await request(app)
    .get(`/api/books/${bookId}`);

  expect(getResponse.status).toBe(200);
  expect(getResponse.body).toMatchObject({
    id: bookId,
    title: 'Clean Code',
    author: 'Robert C. Martin',
    price: 34.99,
    stock: 5
  });
});
```

**Assertions:**
1. HTTP status code is 200
2. Response body matches created book (all fields: id, title, author, price, stock)

**Traceability:**
- Maps to AC-3 part 2 directly
- Maps to breakdown verification step 2: "GET /api/books/:id returns that book"

---

#### Test 3: AC-4 — Shopper POST /api/books returns 403

**Acceptance Criterion:**
> **Given** a shopper request with `x-role: shopper` header  
> **When** they POST `/api/books`  
> **Then** the response is 403 with `{ "error": "<message>" }`

**Test Implementation:**
```javascript
test('AC-4: Shopper POST /api/books returns 403', async () => {
  const bookData = {
    title: 'Forbidden Book',
    author: 'Unauthorized',
    price: 9.99,
    stock: 1
  };

  const response = await request(app)
    .post('/api/books')
    .set('x-role', 'shopper')
    .send(bookData);

  expect(response.status).toBe(403);
  expect(response.body).toEqual({ error: 'Admin role required' });
});
```

**Assertions:**
1. HTTP status code is 403
2. Response body is exactly `{ "error": "Admin role required" }`

**Traceability:**
- Maps to AC-4 directly
- Maps to breakdown verification step 3: "Shopper POST → 403"

---

#### Test 4: No x-role header returns 403

**Verification Step (not a new AC):**
> POST `/api/books` with no `x-role` header → 403

**Test Implementation:**
```javascript
test('No x-role header returns 403', async () => {
  const bookData = {
    title: 'No Role Book',
    author: 'Anonymous',
    price: 5.00,
    stock: 1
  };

  const response = await request(app)
    .post('/api/books')
    .send(bookData);

  expect(response.status).toBe(403);
  expect(response.body).toEqual({ error: 'Admin role required' });
});
```

**Assertions:**
1. HTTP status code is 403
2. Response body is `{ "error": "Admin role required" }`

**Traceability:**
- Maps to breakdown verification step 4: "No x-role header → 403"
- Clarifies AC-4 edge case (no role vs wrong role)

---

#### Test 5: Empty title returns 400

**Verification Step (not a new AC):**
> POST `/api/books` with empty title → 400

**Test Implementation:**
```javascript
test('Empty title returns 400', async () => {
  const bookData = {
    title: '',
    author: 'Some Author',
    price: 19.99,
    stock: 3
  };

  const response = await request(app)
    .post('/api/books')
    .set('x-role', 'admin')
    .send(bookData);

  expect(response.status).toBe(400);
  expect(response.body).toEqual({ error: 'Title is required' });
});
```

**Assertions:**
1. HTTP status code is 400
2. Response body is `{ "error": "Title is required" }`

**Traceability:**
- Maps to breakdown verification step 5: "Empty title → 400"
- Validates REQ-4 (catalog endpoints) validation rules

---

#### Test 6: Price = 0 returns 400

**Verification Step (not a new AC):**
> POST `/api/books` with price = 0 → 400

**Test Implementation:**
```javascript
test('Price = 0 returns 400', async () => {
  const bookData = {
    title: 'Free Book',
    author: 'Giveaway',
    price: 0,
    stock: 100
  };

  const response = await request(app)
    .post('/api/books')
    .set('x-role', 'admin')
    .send(bookData);

  expect(response.status).toBe(400);
  expect(response.body).toEqual({ error: 'Price must be greater than 0' });
});
```

**Assertions:**
1. HTTP status code is 400
2. Response body is `{ "error": "Price must be greater than 0" }`

**Traceability:**
- Maps to breakdown verification step 6: "Price = 0 → 400"
- Validates REQ-4 validation rules (price > 0)

---

#### Test 7: Negative stock returns 400

**Verification Step (not a new AC):**
> POST `/api/books` with stock = -1 → 400

**Test Implementation:**
```javascript
test('Negative stock returns 400', async () => {
  const bookData = {
    title: 'Negative Stock',
    author: 'Error',
    price: 12.00,
    stock: -1
  };

  const response = await request(app)
    .post('/api/books')
    .set('x-role', 'admin')
    .send(bookData);

  expect(response.status).toBe(400);
  expect(response.body).toEqual({ error: 'Stock must be a non-negative integer' });
});
```

**Assertions:**
1. HTTP status code is 400
2. Response body is `{ "error": "Stock must be a non-negative integer" }`

**Traceability:**
- Maps to breakdown verification step 7: "Negative stock → 400"
- Validates REQ-4 validation rules (stock ≥ 0)

---

#### Test 8: AC-5 — GET /api/books returns array of all books

**Acceptance Criterion:**
> **Given** books exist in the catalog  
> **When** a shopper GET `/api/books`  
> **Then** the response is 200 with an array of books, each containing `id`, `title`, `author`, `price`, and `stock`

**Test Implementation:**
```javascript
test('AC-5: GET /api/books returns array of all books', async () => {
  await request(app)
    .post('/api/books')
    .set('x-role', 'admin')
    .send({ title: 'Book 1', author: 'Author A', price: 10.00, stock: 5 });

  await request(app)
    .post('/api/books')
    .set('x-role', 'admin')
    .send({ title: 'Book 2', author: 'Author B', price: 15.00, stock: 3 });

  const response = await request(app).get('/api/books');

  expect(response.status).toBe(200);
  expect(response.body).toHaveLength(2);
  expect(response.body[0]).toHaveProperty('id');
  expect(response.body[0]).toHaveProperty('title');
  expect(response.body[0]).toHaveProperty('author');
  expect(response.body[0]).toHaveProperty('price');
  expect(response.body[0]).toHaveProperty('stock');
});
```

**Assertions:**
1. HTTP status code is 200
2. Response body is an array of length 2
3. Each book has all required fields: id, title, author, price, stock

**Traceability:**
- Maps to AC-5 directly
- Maps to breakdown verification step 8: "GET /api/books → 200 with array"

---

#### Test 9: GET /api/books with no books returns empty array

**Verification Step (not a new AC):**
> GET `/api/books` with empty catalog → 200, empty array

**Test Implementation:**
```javascript
test('GET /api/books with no books returns empty array', async () => {
  const response = await request(app).get('/api/books');

  expect(response.status).toBe(200);
  expect(response.body).toEqual([]);
});
```

**Assertions:**
1. HTTP status code is 200
2. Response body is `[]` (empty array)

**Traceability:**
- Maps to breakdown verification step 9: "No books → 200, empty array"
- Clarifies AC-5 edge case (empty catalog)

---

#### Test 10: GET /api/books/999 (nonexistent) returns 404

**Verification Step (not a new AC):**
> GET `/api/books/:id` for nonexistent book → 404

**Test Implementation:**
```javascript
test('GET /api/books/999 (nonexistent) returns 404', async () => {
  const response = await request(app).get('/api/books/999');

  expect(response.status).toBe(404);
  expect(response.body).toEqual({ error: 'Book not found' });
});
```

**Assertions:**
1. HTTP status code is 404
2. Response body is `{ "error": "Book not found" }`

**Traceability:**
- Maps to breakdown verification step 10: "Nonexistent book → 404"
- Validates REQ-10 error handling (not-found case)

---

#### Test 11: Create 3 books, GET /api/books returns all 3

**Verification Step (not a new AC):**
> Create 3 books, GET `/api/books` → array length 3, all fields present, ordered by ID

**Test Implementation:**
```javascript
test('Create 3 books, GET /api/books returns all 3', async () => {
  await request(app)
    .post('/api/books')
    .set('x-role', 'admin')
    .send({ title: 'Book A', author: 'Author 1', price: 10.00, stock: 2 });

  await request(app)
    .post('/api/books')
    .set('x-role', 'admin')
    .send({ title: 'Book B', author: 'Author 2', price: 20.00, stock: 4 });

  await request(app)
    .post('/api/books')
    .set('x-role', 'admin')
    .send({ title: 'Book C', author: 'Author 3', price: 30.00, stock: 6 });

  const response = await request(app).get('/api/books');

  expect(response.status).toBe(200);
  expect(response.body).toHaveLength(3);
  expect(response.body[0].title).toBe('Book A');
  expect(response.body[1].title).toBe('Book B');
  expect(response.body[2].title).toBe('Book C');
});
```

**Assertions:**
1. HTTP status code is 200
2. Response body array length is 3
3. Books are ordered by ID ascending (verified via title order)

**Traceability:**
- Maps to breakdown verification step 11: "Create 3 books, list all"
- Validates AC-5 ordering (ORDER BY id ASC)

---

## Complete Test File

**File:** `__tests__/books.test.js`

```javascript
const request = require('supertest');
const app = require('../src/app');
const { db } = require('../src/db');

describe('Books API and role enforcement', () => {
  beforeEach(() => {
    // Clear product table before each test
    db.prepare('DELETE FROM product').run();
  });

  afterAll(() => {
    db.close();
  });

  test('AC-3 part 1: Admin POST /api/books creates book and returns 201', async () => {
    const bookData = {
      title: 'The Pragmatic Programmer',
      author: 'Hunt & Thomas',
      price: 29.99,
      stock: 10
    };

    const response = await request(app)
      .post('/api/books')
      .set('x-role', 'admin')
      .send(bookData);

    expect(response.status).toBe(201);
    expect(response.body).toMatchObject({
      id: expect.any(Number),
      title: 'The Pragmatic Programmer',
      author: 'Hunt & Thomas',
      price: 29.99,
      stock: 10
    });
  });

  test('AC-3 part 2: GET /api/books/:id returns created book', async () => {
    const bookData = {
      title: 'Clean Code',
      author: 'Robert C. Martin',
      price: 34.99,
      stock: 5
    };

    const createResponse = await request(app)
      .post('/api/books')
      .set('x-role', 'admin')
      .send(bookData);

    const bookId = createResponse.body.id;

    const getResponse = await request(app)
      .get(`/api/books/${bookId}`);

    expect(getResponse.status).toBe(200);
    expect(getResponse.body).toMatchObject({
      id: bookId,
      title: 'Clean Code',
      author: 'Robert C. Martin',
      price: 34.99,
      stock: 5
    });
  });

  test('AC-4: Shopper POST /api/books returns 403', async () => {
    const bookData = {
      title: 'Forbidden Book',
      author: 'Unauthorized',
      price: 9.99,
      stock: 1
    };

    const response = await request(app)
      .post('/api/books')
      .set('x-role', 'shopper')
      .send(bookData);

    expect(response.status).toBe(403);
    expect(response.body).toEqual({ error: 'Admin role required' });
  });

  test('No x-role header returns 403', async () => {
    const bookData = {
      title: 'No Role Book',
      author: 'Anonymous',
      price: 5.00,
      stock: 1
    };

    const response = await request(app)
      .post('/api/books')
      .send(bookData);

    expect(response.status).toBe(403);
    expect(response.body).toEqual({ error: 'Admin role required' });
  });

  test('Empty title returns 400', async () => {
    const bookData = {
      title: '',
      author: 'Some Author',
      price: 19.99,
      stock: 3
    };

    const response = await request(app)
      .post('/api/books')
      .set('x-role', 'admin')
      .send(bookData);

    expect(response.status).toBe(400);
    expect(response.body).toEqual({ error: 'Title is required' });
  });

  test('Price = 0 returns 400', async () => {
    const bookData = {
      title: 'Free Book',
      author: 'Giveaway',
      price: 0,
      stock: 100
    };

    const response = await request(app)
      .post('/api/books')
      .set('x-role', 'admin')
      .send(bookData);

    expect(response.status).toBe(400);
    expect(response.body).toEqual({ error: 'Price must be greater than 0' });
  });

  test('Negative stock returns 400', async () => {
    const bookData = {
      title: 'Negative Stock',
      author: 'Error',
      price: 12.00,
      stock: -1
    };

    const response = await request(app)
      .post('/api/books')
      .set('x-role', 'admin')
      .send(bookData);

    expect(response.status).toBe(400);
    expect(response.body).toEqual({ error: 'Stock must be a non-negative integer' });
  });

  test('AC-5: GET /api/books returns array of all books', async () => {
    await request(app)
      .post('/api/books')
      .set('x-role', 'admin')
      .send({ title: 'Book 1', author: 'Author A', price: 10.00, stock: 5 });

    await request(app)
      .post('/api/books')
      .set('x-role', 'admin')
      .send({ title: 'Book 2', author: 'Author B', price: 15.00, stock: 3 });

    const response = await request(app).get('/api/books');

    expect(response.status).toBe(200);
    expect(response.body).toHaveLength(2);
    expect(response.body[0]).toHaveProperty('id');
    expect(response.body[0]).toHaveProperty('title');
    expect(response.body[0]).toHaveProperty('author');
    expect(response.body[0]).toHaveProperty('price');
    expect(response.body[0]).toHaveProperty('stock');
  });

  test('GET /api/books with no books returns empty array', async () => {
    const response = await request(app).get('/api/books');

    expect(response.status).toBe(200);
    expect(response.body).toEqual([]);
  });

  test('GET /api/books/999 (nonexistent) returns 404', async () => {
    const response = await request(app).get('/api/books/999');

    expect(response.status).toBe(404);
    expect(response.body).toEqual({ error: 'Book not found' });
  });

  test('Create 3 books, GET /api/books returns all 3', async () => {
    await request(app)
      .post('/api/books')
      .set('x-role', 'admin')
      .send({ title: 'Book A', author: 'Author 1', price: 10.00, stock: 2 });

    await request(app)
      .post('/api/books')
      .set('x-role', 'admin')
      .send({ title: 'Book B', author: 'Author 2', price: 20.00, stock: 4 });

    await request(app)
      .post('/api/books')
      .set('x-role', 'admin')
      .send({ title: 'Book C', author: 'Author 3', price: 30.00, stock: 6 });

    const response = await request(app).get('/api/books');

    expect(response.status).toBe(200);
    expect(response.body).toHaveLength(3);
    expect(response.body[0].title).toBe('Book A');
    expect(response.body[1].title).toBe('Book B');
    expect(response.body[2].title).toBe('Book C');
  });
});
```

---

## Test Execution

### Running Tests

```bash
# Install dependencies (first time only)
npm install

# Run all tests
npm test

# Run with verbose output
npm test -- --verbose

# Run only books tests
npm test -- books.test.js

# Run with coverage
npm test -- --coverage
```

### Expected Output

```
PASS  __tests__/books.test.js
  Books API and role enforcement
    ✓ AC-3 part 1: Admin POST /api/books creates book and returns 201 (XX ms)
    ✓ AC-3 part 2: GET /api/books/:id returns created book (XX ms)
    ✓ AC-4: Shopper POST /api/books returns 403 (XX ms)
    ✓ No x-role header returns 403 (XX ms)
    ✓ Empty title returns 400 (XX ms)
    ✓ Price = 0 returns 400 (XX ms)
    ✓ Negative stock returns 400 (XX ms)
    ✓ AC-5: GET /api/books returns array of all books (XX ms)
    ✓ GET /api/books with no books returns empty array (XX ms)
    ✓ GET /api/books/999 (nonexistent) returns 404 (XX ms)
    ✓ Create 3 books, GET /api/books returns all 3 (XX ms)

Test Suites: 1 passed, 1 total
Tests:       11 passed, 11 total
```

---

## Verification Checklist

Use this checklist to verify test implementation matches the breakdown:

- [x] **Test Count:** 11 tests implemented (matches breakdown count exactly)
- [x] **AC-3 Coverage:** Admin create book and retrieve by ID
- [x] **AC-4 Coverage:** Shopper 403 on POST
- [x] **AC-5 Coverage:** List all books with required fields
- [x] **Validation Coverage:** Empty title, price = 0, negative stock (all return 400)
- [x] **Auth Coverage:** No x-role header returns 403
- [x] **Not-found Coverage:** GET nonexistent book returns 404
- [x] **Edge Cases:** Empty catalog, multiple books, ordering by ID
- [x] **File Paths:** `__tests__/books.test.js` matches LLD specification
- [x] **Test Framework:** Jest + Supertest as specified in repo-context
- [x] **No Extra Tests:** No tests added beyond breakdown verification approach
- [x] **Traceability:** Each test explicitly references its AC or verification step

---

## Consistency Validation

### Breakdown ↔ Tests Alignment

| Breakdown Verification Step | Test Implementation | Status |
|-----------------------------|---------------------|--------|
| 1. Admin POST with valid data → 201 | Test 1 | ✓ |
| 2. GET /api/books/:id returns book | Test 2 | ✓ |
| 3. Shopper POST → 403 | Test 3 | ✓ |
| 4. No x-role header → 403 | Test 4 | ✓ |
| 5. Empty title → 400 | Test 5 | ✓ |
| 6. Price = 0 → 400 | Test 6 | ✓ |
| 7. Negative stock → 400 | Test 7 | ✓ |
| 8. GET /api/books → 200 with array | Test 8 | ✓ |
| 9. No books → 200, empty array | Test 9 | ✓ |
| 10. Nonexistent book → 404 | Test 10 | ✓ |
| 11. Create 3 books, list all | Test 11 | ✓ |

### Requirements ↔ Tests Alignment

| Requirement | Test Coverage | Status |
|-------------|---------------|--------|
| REQ-3: Role-based access control | Tests 3, 4 (admin/shopper/no-role) | ✓ |
| REQ-4: Catalog endpoints (POST, GET list, GET by ID) | Tests 1, 2, 8, 9, 10, 11 | ✓ |
| REQ-10: Error handling (403, 400, 404) | Tests 3-7, 10 | ✓ |

### HLD ↔ Tests Alignment

| HLD Component | Test Verification | Status |
|---------------|-------------------|--------|
| src/middleware/role.js (roleReader, requireAdmin) | Tests 3, 4 (403 enforcement) | ✓ |
| src/routes/books.js (POST /, GET /, GET /:id) | All 11 tests | ✓ |
| Validation rules (title, author, price, stock) | Tests 5, 6, 7 | ✓ |
| Error responses (403, 400, 404) | Tests 3-7, 10 | ✓ |
| Database operations (INSERT, SELECT) | Tests 1, 2, 8, 11 | ✓ |

---

## Notes

- **No source code review:** This document generates tests only, per the `bmad-qa-generate-e2e-tests` skill scope.
- **No extra tests:** Test count (11) matches breakdown verification approach count exactly. Validation and edge case tests are all listed in the breakdown; no new cases invented.
- **File paths match repo-tree:** All paths resolve correctly from `/home/pavc/Documents/Book-Nest-Product/`.
- **Stack alignment:** Tests use Jest and Supertest as specified in repo-context for the Book repository. No TypeScript, no Playwright (plain JavaScript as required).
- **Test isolation:** `beforeEach` deletes all `product` rows to ensure clean state between tests. This is sufficient for Story 2 (only one table touched).
- **ADLC consistency:** No acceptance criteria invented; all ACs come from breakdown/requirements. Test count matches verification approach. File paths match LLD and repo-tree.
- **Error message precision:** Tests check exact error messages (e.g. "Title is required" vs "Admin role required") to ensure clear API contract.
- **Ordering verification:** Test 11 verifies books are ordered by ID ascending (GET /api/books returns books in creation order).

---

## Appendix: Test Data Requirements

### Test Isolation Strategy

**Setup (beforeEach):**
- Delete all rows from `product` table: `db.prepare('DELETE FROM product').run();`
- This ensures each test starts with an empty catalog.

**Teardown (afterAll):**
- Close database connection: `db.close();`
- Prevents file lock issues when running full test suite.

### Sample Book Data

Tests use realistic book data to improve readability:

- **Test 1:** "The Pragmatic Programmer" by Hunt & Thomas, $29.99, stock 10
- **Test 2:** "Clean Code" by Robert C. Martin, $34.99, stock 5
- **Test 3:** "Forbidden Book" by Unauthorized, $9.99, stock 1 (shopper attempt)
- **Test 5:** Empty title → validation failure
- **Test 6:** "Free Book" by Giveaway, price 0 → validation failure
- **Test 7:** "Negative Stock" by Error, stock -1 → validation failure
- **Tests 8, 11:** Generic "Book A/B/C" with sequential prices and stock

**No Data Seeding Required:**
- All tests create their own data via POST `/api/books`.
- No external seed files or fixtures.
- `beforeEach` cleanup ensures no data leaks between tests.

---

## Summary

✅ **11 tests implemented** (matches breakdown count)  
✅ **AC-3, AC-4, AC-5 covered**  
✅ **All verification steps mapped to test assertions**  
✅ **File paths match LLD and repo-tree**  
✅ **Stack matches repo-context (Jest, Supertest, CommonJS)**  
✅ **No invented ACs or extra tests beyond breakdown**  
✅ **ADLC consistency validated**

**Test suite ready for execution.**
