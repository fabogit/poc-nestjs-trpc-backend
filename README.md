# poc-nestjs-trpc-backend

> A modern type-safe API backend integrating tRPC v10 into NestJS 10 with automatic schema reflection, runtime Zod validation, request observability middleware, and interactive API exploration via tRPC Panel.

**Inception (First Commit):** `2025-01-28` • **Batch / Horizon:** `Batch 2024–2025` • **Architecture Domain:** `End-to-End Type-Safe RPC Backend`

---

## 📌 Architecture & Overview

`nestjs-tRPC` is an architectural demonstration combining the robust modular dependency injection of **NestJS** with the end-to-end compile-time type safety of **tRPC**. Built on top of `nestjs-trpc` (v1.6) and Express, the application exposes Remote Procedure Call (RPC) interfaces defined through TypeScript decorators while automatically generating client contracts without requiring manual schema synchronizations.

```
src/
├── main.ts                        # Application entrypoint & HTTP server bootstrap (Port 3000)
├── app.module.ts                  # Root application module wiring TrpcModule and ProductsModule
├── products/
│   ├── product.schema.ts          # Zod schema definitions and inferred TypeScript domain types
│   ├── products.module.ts         # Products feature module registering Service and Router
│   ├── products.router.ts         # tRPC Router handling Query and Mutation procedures
│   └── products.service.ts        # Business logic & in-memory state repository
└── trpc/
    ├── trpc.module.ts             # Global tRPC configuration, schema auto-generation & middleware
    ├── trpc-panel.controller.ts   # Express controller serving the trpc-panel documentation UI
    ├── @generated/
    │   └── server.ts              # Automatically generated standalone tRPC router contract
    ├── context/
    │   ├── app.context.ts         # TRPCContext provider extracting Express Request & Response
    │   └── context.interface.ts   # Context typing interface definition
    └── middleware/
        └── logger.middleware.ts   # Procedural latency, IP, method, and status code logging middleware
```

* **Context:** Architectural proof-of-concept evaluating `nestjs-trpc` as an alternative to REST and GraphQL for full-stack TypeScript applications, eliminating code generation steps and enabling end-to-end type safety between backend services and frontend clients.
* **Key Patterns:**
  * **Decorator-Driven RPC Routing:** Procedures are defined using `@Router`, `@Query`, and `@Mutation` decorators directly within NestJS injectable classes (`ProductsRouter`).
  * **Automated Contract Generation:** Configured with `autoSchemaFile: 'src/trpc/@generated'`, compiling NestJS router classes into standard `@trpc/server` router definitions (`src/trpc/@generated/server.ts`) for frontend client export.
  * **Scoped Middleware Pipeline:** Procedural middleware (`LoggerMiddleware`) intercepting execution chains, tracking duration in milliseconds, capturing client IP and response status codes.
  * **Context Injection:** Encapsulation of native Express request and response lifecycle objects inside `AppContext` passed down to procedures via `@Ctx()`.
  * **Interactive Explorer:** Integrated `trpc-panel` at `/panel` allowing visual testing and inspection of tRPC queries and mutations directly in the browser.

---

## 🛠 Tech Stack

* **Runtime & Framework:** Node.js (>= 18), TypeScript (ES2024 target), NestJS 10 (`@nestjs/core`, `@nestjs/common`, `@nestjs/platform-express`), `pnpm`
* **RPC & Validation:** `nestjs-trpc` 1.6.1, `@trpc/server` 10.45.2, `zod` 3.24.1
* **Developer Tooling & UI:** `trpc-panel` 1.3.4
* **State Management:** In-memory collection (`Product[]`) within `ProductsService`
* **Testing & Quality:** Jest 29, `ts-jest`, Supertest 7, ESLint, Prettier

---

## ✨ Features & Endpoints

* **Full CRUD Product Management:** In-memory procedural workflows to fetch, create, partially update, and delete product catalog items.
* **Runtime Input & Output Validation:** Strict schema validation via Zod; invalid inputs automatically return formatted tRPC validation errors (`BAD_REQUEST`).
* **Observable Procedure Execution:** Execution logs capture latency (`durationMs`), route path, call type (`query` vs `mutation`), HTTP method, and response status.
* **Interactive Documentation UI:** Web UI available at `http://localhost:3000/panel` to execute queries and mutations without external client setup.

### Key API Routes & Procedures

#### tRPC Procedure Interface (`/trpc`)

The primary API interface is served over the tRPC transport endpoint (`/trpc/<procedurePath>`). Queries are accessed via HTTP `GET` with encoded query parameters, while Mutations are dispatched via HTTP `POST`.

| Procedure Path | RPC Type | Input Schema | Output Schema | Description |
|---|---|---|---|---|
| `products.getAllProducts` | `Query` | _None_ (`void`) | `Product[]` | Retrieves all product items stored in memory. |
| `products.getProductById` | `Query` | `{ id: string }` | `Product` | Retrieves a specific product by ID; throws `NOT_FOUND` if missing. |
| `products.createProduct` | `Mutation` | `Product` | `Product` | Appends a new product item to the collection. |
| `products.updateProduct` | `Mutation` | `{ id: string, data: Partial<Product> }` | `Product` | Partially updates fields on an existing product; throws `NOT_FOUND` if missing. |
| `products.deleteProduct` | `Mutation` | `{ id: string }` | `boolean` | Deletes a product by ID; returns `true` on success or `false` if not found. |

#### HTTP Endpoints

| Method | Route | Output | Description |
|---|---|---|---|
| `ALL` / `GET` | `/panel` | `text/html` | Serves the interactive `trpc-panel` UI for executing procedures in browser. |
| `ALL` | `/trpc/*` | `application/json` | Base HTTP endpoint mounting the tRPC query and mutation handlers. |

> **Note on Data Schema:** In `src/products/product.schema.ts`, the nested property under `details` is defined as `desciption` (notice spelling). The input validation strictly expects this key during `createProduct` and `updateProduct`.

---

## 🚀 Getting Started

### Prerequisites

* **Node.js** `>= 18.0.0`
* **pnpm** `>= 8.0.0` (or `npm` / `yarn`)

### Installation

```bash
# Clone the repository
git clone https://github.com/fabogit/poc-nestjs-trpc-backend.git
cd poc-nestjs-trpc-backend

# Install dependencies using pnpm
pnpm install
```

### Environment Configuration

The application defaults to port `3000`. To customize the listening port, create an optional `.env` file or export `PORT`:

```bash
PORT=3000
```

### Running the Application

```bash
# Development mode with hot-reload
pnpm run start:dev

# Standard start
pnpm run start

# Production build and run
pnpm run build
pnpm run start:prod
```

Once running, the application will be available at:
* **tRPC Panel Explorer:** [http://localhost:3000/panel](http://localhost:3000/panel)
* **tRPC Endpoint Base:** [http://localhost:3000/trpc](http://localhost:3000/trpc)

---

## 🧪 Testing & Verification

### Interacting via tRPC Panel

1. Start the server using `pnpm run start:dev`.
2. Open [http://localhost:3000/panel](http://localhost:3000/panel) in your browser.
3. Select `products.createProduct` mutation, provide a JSON payload, and execute:
   ```json
   {
     "id": "prod-101",
     "name": "Mechanical Keyboard",
     "price": 149.99,
     "details": {
       "desciption": "RGB Hot-swappable keyboard",
       "rating": 5
     }
   }
   ```
4. Query all stored products using `products.getAllProducts`.

### Interacting via cURL

Because tRPC maps procedures over standard HTTP, procedures can be queried directly:

```bash
# Create a product (Mutation)
curl -X POST http://localhost:3000/trpc/products.createProduct \
  -H "Content-Type: application/json" \
  -d '{"id":"prod-1","name":"Wireless Mouse","price":49.99,"details":{"desciption":"Ergonomic mouse","rating":4}}'

# Fetch product by ID (Query - URL encoded input)
curl -G http://localhost:3000/trpc/products.getProductById \
  --data-urlencode 'input={"id":"prod-1"}'

# Fetch all products (Query)
curl http://localhost:3000/trpc/products.getAllProducts
```

### Automated Tests

```bash
# Unit tests
pnpm run test

# Test coverage
pnpm run test:cov
```

> **Testing Caveat:** The repository contains the default starter e2e test (`test/app.e2e-spec.ts`) asserting `GET /` against an obsolete `AppController`. Because `AppController` was removed in favor of tRPC routing, running `pnpm run test:e2e` yields a 404 response until updated to target tRPC procedures. See the accompanying critique report for a drop-in tRPC E2E test fixture.

---

## 📄 License & Author

Developed by [@fabogit](https://github.com/fabogit).
