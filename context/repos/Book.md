# Book

Node.js HTTP API for the BookNest bookstore. This repo starts empty. The first ticket creates the whole application.

## Stack

- Language: Node.js 20, plain JavaScript (CommonJS).
- HTTP: Express. JSON REST under `/api`.
- Database: SQLite via better-sqlite3. Database file: `data/booknest.sqlite`.
- Tests: Jest, `npm test`.
- Server port: `PORT`, default 3000.

## Do

- Create `package.json`, the schema, `src/server.js`, routes, and tests.
- Create the tables on startup if they are missing.
- Treat `x-role: shopper` and `x-role: admin` as the only identity check.
- Identify a cart with the `x-cart-id` header.
- Return errors as `{ "error": "<message>" }` with the correct HTTP status.

## Don't

- Do not add a frontend, React, HTML, or CSS.
- Do not add Postgres, Docker, Prisma, Redis, or a real payment provider.
- Do not add user registration, passwords, or OAuth.
- Do not add TypeScript.