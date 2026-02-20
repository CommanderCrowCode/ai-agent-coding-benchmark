# Relative Grading Report: Emergent

## Summary
- **Weighted Average: 5.75/10**
- **Grade: C**
- **Interpretation:** Functional core business logic but significant issues — no tests, no infrastructure, broken deployment, non-functional frontend

## Critical Findings

### Architecture Deviations from Spec
1. **MongoDB instead of SQLAlchemy/PostgreSQL** — The entire backend uses MongoDB (motor/pymongo) instead of the specified SQLAlchemy ORM with PostgreSQL. No formal models, no migrations, no enum types, no FK constraints.
2. **React CRA instead of Next.js 14** — Frontend is Create React App (with craco) instead of Next.js 14 App Router. No `page.tsx`, no `layout.tsx`, no SSR.
3. **Broken imports** — `server.py` imports from `routers.auth`, `routers.products`, etc., but the `routers/` directory is **empty**. The actual router files are at the backend root level. The backend **cannot start** as-is.
4. **No pyproject.toml / uv** — Uses `requirements.txt` instead of `pyproject.toml` with `uv`.

---

## Feature Scores

### F01: Project Structure & Setup (Weight: 5%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| Directory structure matches spec | 2 | 0 | Router files at root instead of `routers/`. No `schemas/`, `utils/`, `tasks/`, `alembic/` dirs. `routers/` dir exists but is empty |
| `pyproject.toml` with all dependencies | 2 | 0 | Uses `requirements.txt` (`backend/requirements.txt:1-131`), not `pyproject.toml` |
| `alembic.ini` + alembic directory | 1 | 0 | Missing entirely — uses MongoDB, no SQL migrations |
| `uv` used for package management | 1 | 0 | No `uv.lock` or any evidence of `uv` usage |
| Frontend `package.json` with correct deps | 1 | 0 | React 19 + CRA (`frontend/package.json:44`), NOT Next.js 14 + React 18 |
| `tsconfig.json` and `tailwind.config.ts` | 1 | 0.5 | Has `jsconfig.json` (not tsconfig) and `tailwind.config.js` (not .ts) |
| `.env` example or config documentation | 1 | 1 | `.env` present in both `backend/.env` and `frontend/.env` |
| Clean separation of concerns | 1 | 0.5 | Intended routers/services split exists conceptually but broken (files in wrong directory) |

**Feature Score: 2/10**

---

### F02: Database Schema & Models (Weight: 10%)

**Note:** Uses MongoDB collections with implicit schemas instead of SQLAlchemy models. Scoring based on document structure and fields used across the codebase.

#### F02a: Auth & Security Tables

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `users` table — all fields | 2 | 1.5 | Has id, email, password_hash, name, role, is_active, mfa_enabled, created_at (`services/seed.py:22-31`). Missing: `mfa_secret` field never stored |
| `audit_log` table — all fields including JSON old/new values | 2 | 2 | Full fields: id, table_name, record_id, action, old_values (dict), new_values (dict), user_id, ip_address, user_agent, created_at (`services/audit.py:30-41`) |

#### F02b: Products & Suppliers Tables

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `suppliers` — all fields | 1 | 1 | Complete: name, legal_name, website, address, payment_terms, lead_time_days, moq, notes (`suppliers.py:18-27`) |
| `supplier_contacts` — FK to supplier, platform | 1 | 1 | Has supplier_id, platform field (`suppliers.py:40-44`) |
| `products` — all fields with constraints | 2 | 1.5 | Has internal_sku, supplier_sku, age ranges, price_usd, units_per_carton, is_best_seller, is_active, supplier_id. unique index on internal_sku (`server.py:47`). No SKU format validation |
| `price_history` — product_id, supplier_id, price_usd, effective_date | 1 | 1 | Complete (`products.py:121-127`) |

#### F02c: Purchase Orders Tables

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `purchase_orders` — all fields with constraints | 2 | 1.5 | Has po_number, status, financial fields. Status is plain string, not enum (`orders.py:153-166`) |
| `po_line_items` — FK to po and product | 1 | 1 | po_id, product_id, quantity, unit_price_usd (`orders.py:172-178`) |
| `po_status_history` — FK to po, old/new status | 1 | 1 | Complete with changed_by, changed_at, notes (`orders.py:181-189`) |

#### F02d: Shipments Tables

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `shipments` table | 1 | 0 | **Not implemented** |
| `shipment_milestones` table | 1 | 0 | **Not implemented** |

#### F02e: Payments Table

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `payments` — all fields | 2 | 1.5 | Has payment_number (PAY-YYYY-NNN), po_id, supplier_id, amount_usd, fee_usd, status, amount_received, currency_received. Missing some Wise-specific fields (`payments.py:132-146`) |

#### F02f: Inventory Tables

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `inventory` — all fields with defaults | 2 | 2 | product_id (unique index), quantity, reorder_point (default 10), safety_stock_days (default 14), last_counted_at (`inventory.py:130-137`, `server.py:58`) |
| `stock_movements` — movement_type enum, reference_type | 1 | 1 | IN/OUT/ADJUSTMENT types, reference_type, reference_id (`inventory.py:205-215`) |
| `sales` — product_id, platform enum, order_reference | 1 | 1 | Platform: SHOPEE/LAZADA/DIRECT, order_reference present (`inventory.py:269-278`) |

#### F02g: Planning Tables

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `seasonality_factors` | 1 | 0 | **Not implemented** |
| `holidays` | 1 | 0 | **Not implemented** |
| `settings` | 0.5 | 0 | **Not implemented** |
| `notifications` | 0.5 | 0 | Indexed (`server.py:55`) but never populated or used in code |

#### F02h: Indexes & Constraints

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| All specified indexes (11 total) | 3 | 3 | 13 indexes created in `server.py:47-59` — exceeds requirement |
| Unique constraints on internal_sku, po_number, payment_id | 2 | 1 | Only `internal_sku` has unique index (`server.py:47`). po_number and payment_number use regex-based generation without unique constraint |
| Foreign key relationships correctly defined | 2 | 1 | MongoDB has no FK enforcement. Manual lookups exist in code but no referential integrity |
| Proper use of enums for status fields | 1 | 0 | All status fields are plain strings. PO_STATUSES is a Python list (`orders.py:18-21`), not an enum type |
| Table count >= 20 | 2 | 0 | 14 collections used (users, audit_log, products, suppliers, supplier_contacts, price_history, purchase_orders, po_line_items, po_status_history, payments, inventory, stock_movements, sales, notifications). Missing 6+ collections |

**Subtotal: 22/30 → Normalized: 7.3/10**

**Feature Score: 7.3/10**

---

### F03: Application Setup & Configuration (Weight: 5%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| FastAPI app with lifespan manager | 1 | 1 | `@asynccontextmanager` lifespan creates indexes and seeds DB on startup (`server.py:32-43`) |
| CORS configured for specified origins | 1 | 0 | `CORS_ORIGINS="*"` in `backend/.env:3`. Wildcard CORS |
| Global rate limiter: 100 req/min | 1 | 0 | `slowapi` in `requirements.txt:109` but **never imported or used** in code |
| Auth endpoint rate limit: 5 req/min | 1 | 0 | **Not implemented** |
| Security headers middleware (all 4) | 2 | 1 | 3 of 4 present: X-Content-Type-Options, X-Frame-Options, Referrer-Policy (`server.py:66-71`). **Missing: Content-Security-Policy** |
| Health check GET /health | 1 | 0.5 | Returns `{"status": "ok"}` but at `/api/health` not `/health` (`server.py:88-90`) |
| Pydantic Settings with all env vars | 2 | 0 | Uses `os.environ` directly (`server.py:24-26`), no Pydantic Settings class |
| `app/database.py` with get_db dependency | 1 | 0.5 | `get_db()` exists in `server.py:83-84` but returns MongoDB client directly, not a proper session dependency |

**Feature Score: 3/10**

---

### F04: Auth Router (Weight: 8%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| POST /api/auth/register — bcrypt hash | 2 | 2 | Creates user with `bcrypt.gensalt(rounds=12)` (`auth.py:47`), returns JWT tokens (`auth.py:91-119`) |
| POST /api/auth/login — access + refresh JWT | 2 | 2 | Returns both tokens correctly (`auth.py:122-139`) |
| POST /api/auth/refresh — exchange tokens | 1 | 1 | Validates refresh token type, returns new access token (`auth.py:142-159`) |
| POST /api/auth/mfa/setup — TOTP + QR URI | 1.5 | 0 | **Not implemented.** `pyotp` in requirements but never imported or used |
| POST /api/auth/mfa/verify — validates TOTP | 1.5 | 0 | **Not implemented** |
| JWT HS256 with 15-min access | 1 | 1 | `ALGORITHM = "HS256"`, `ACCESS_TOKEN_EXPIRE_MINUTES = 15` (`auth.py:13-14`) |
| Refresh token: 7-day expiry | 0.5 | 0.5 | `REFRESH_TOKEN_EXPIRE_DAYS = 7` (`auth.py:15`) |
| Rate limiting: 5 req/min on auth | 0.5 | 0 | **Not implemented** |

**Feature Score: 6.5/10**

---

### F05: Products Router (Weight: 7%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| GET /api/products — filtering (best_seller, active) | 2 | 2 | Both filters implemented (`products.py:52-57`) |
| GET /api/products/{id} — with supplier info | 1 | 1 | Fetches supplier relationship (`products.py:76-78`) and price history (`products.py:81-84`) |
| POST /api/products — create + price_history | 2 | 2 | Creates product and records initial price in price_history (`products.py:118-127`) |
| PUT /api/products/{id} — price_history on price change | 2 | 2 | Detects price change with `updates["price_usd"] != product.get("price_usd")` and records (`products.py:160-167`) |
| DELETE /api/products/{id} — soft delete | 1.5 | 1.5 | Sets `is_active: False` (`products.py:184`) |
| Proper Pydantic schemas | 1 | 1 | `ProductCreate` and `ProductUpdate` well-defined (`products.py:18-40`) |
| Input validation (SKU format) | 0.5 | 0 | Only checks uniqueness, no LUMI-xxxx format validation |

**Feature Score: 9.5/10**

---

### F06: Suppliers Router (Weight: 5%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| GET /api/suppliers — list all | 1.5 | 1.5 | Sorted by name (`suppliers.py:47-51`) |
| GET /api/suppliers/{id} — detail with contacts | 2 | 2 | Includes contacts AND products (`suppliers.py:54-64`) |
| GET /api/suppliers/{id}/contacts — contacts only | 1.5 | 1.5 | Dedicated endpoint (`suppliers.py:67-71`) |
| POST /api/suppliers — create | 2 | 2 | With audit logging (`suppliers.py:74-89`) |
| PUT /api/suppliers/{id} — update | 2 | 2 | With change detection and audit logging (`suppliers.py:92-109`) |
| Proper Pydantic schemas | 1 | 1 | `SupplierCreate`, `SupplierUpdate`, `ContactCreate` defined (`suppliers.py:18-44`) |

**Feature Score: 10/10**

---

### F07: Purchase Orders Router (Weight: 10%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| GET /api/orders — filters (status, supplier_id) | 1 | 1 | Both filters implemented (`orders.py:83-87`) |
| GET /api/orders/{id} — line_items + status_history | 1.5 | 1.5 | Both included with product enrichment (`orders.py:100-128`) |
| GET /api/orders/workflow-stats — count per status | 1 | 1 | MongoDB aggregation pipeline (`orders.py:63-73`) |
| POST /api/orders — create with line items | 1 | 1 | Creates order + line items with validation (`orders.py:131-193`) |
| Auto-generate PO-YYYY-NNN | 1 | 1 | Correct format with auto-increment via regex query (`orders.py:49-60`) |
| Calculate totals from line items | 1 | 1 | `subtotal = sum(item.quantity * item.unit_price_usd)` (`orders.py:150`) |
| PUT /api/orders/{id} — update | 0.5 | 0.5 | Updates notes, fees, recalculates total (`orders.py:196-219`) |
| POST /api/orders/{id}/status — advance | 1 | 1 | Works with validation (`orders.py:222-260`) |
| All 10 status stages defined | 1 | 1 | DRAFT, SENT, CONFIRMED, IN_PRODUCTION, SHIPPED, IN_TRANSIT, CUSTOMS, RECEIVED, QCD, STOCKED (`orders.py:18-21`) |
| Forward-only transitions enforced | 1 | 1 | `new_idx <= current_idx` returns 400 error (`orders.py:238-239`) |
| Status change recorded in history | 0.5 | 0.5 | Inserted into `po_status_history` (`orders.py:247-255`) |

**Feature Score: 10/10**

---

### F08: Payments Router (Weight: 7%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| GET /api/payments — list with status filter | 1 | 1 | Filter works (`payments.py:75-85`) |
| GET /api/payments/{id} — detail | 1 | 1 | Includes supplier and PO enrichment (`payments.py:98-112`) |
| GET /api/payments/summary — monthly, YTD, fees | 2 | 1.5 | Has MTD, YTD, total_fees, pending. No per-month breakdown (`payments.py:50-72`) |
| POST /api/payments — auto PAY-YYYY-NNN | 2 | 2 | Correct format with auto-increment (`payments.py:36-47`, `payments.py:115-151`) |
| PUT /api/payments/{id} — update + audit_log | 2 | 2 | Status validation + audit logging (`payments.py:154-174`) |
| GET /api/payments/{id}/sync-wise stub | 1 | 0 | **Not implemented** — no sync-wise endpoint exists |
| Proper Pydantic schemas | 1 | 1 | `PaymentCreate`, `PaymentUpdate` defined (`payments.py:21-33`) |

**Feature Score: 8.5/10**

---

### F09: Inventory Router (Weight: 15%)

#### F09a: Read Endpoints

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| GET /api/inventory/ — list with stock status | 2 | 2 | `determine_status()` correctly computes OK/LOW/OUT_OF_STOCK (`inventory.py:46-51`, `inventory.py:93-138`) |
| Status filter query param | 1 | 1 | `status_filter` parameter works (`inventory.py:121-122`) |
| Uses joinedload for product | 0.5 | 0 | N+1 queries — separate MongoDB query per inventory item (`inventory.py:104-107`) |
| GET /api/inventory/stats — 5 aggregate fields | 1.5 | 1.5 | total_value, total_products, in_stock_count, low_stock_count, out_of_stock_count (`inventory.py:54-90`) |
| GET /api/inventory/{product_id} — auto-creates if missing | 1 | 1 | Creates with qty=0, reorder_point=10 if not found (`inventory.py:149-160`) |
| GET /api/inventory/{product_id}/movements — paginated | 1 | 1 | skip/limit pagination with total count (`inventory.py:173-186`) |

#### F09b: Receive Shipment

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| POST /api/inventory/receive-shipment — bulk items + reference | 1 | 1 | Accepts items list + shipment_reference (`inventory.py:18-25`) |
| Validates ALL products before processing | 1.5 | 1.5 | Pre-validation loop checks all products first (`inventory.py:195-200`) |
| Creates IN stock movement per item | 1 | 1 | Movement created per item (`inventory.py:204-215`) |
| Updates inventory quantity and updated_at | 0.5 | 0.5 | Both updated (`inventory.py:220-224`) |

#### F09c: Sales

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| POST /api/inventory/sale — validates sufficient stock | 1 | 1 | `current_qty < data.quantity` check (`inventory.py:252-253`) |
| Creates OUT movement + sale record | 1 | 1 | Both stock_movement (OUT) and sale record created (`inventory.py:256-278`) |
| Decrements inventory | 0.5 | 0.5 | `new_qty = current_qty - data.quantity` (`inventory.py:281`) |
| POST /api/inventory/bulk-sale — atomic bulk | 1.5 | 1.5 | Full pre-validation of all items before any modification (`inventory.py:296-308`) |
| Insufficient stock returns 400 | 1 | 1 | HTTPException 400 with detail message (`inventory.py:253`) |

#### F09d: Adjustment

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| POST /api/inventory/adjust — absolute target | 1 | 1 | `data.quantity` is target, `delta = data.quantity - current_qty` (`inventory.py:362`) |
| Prevents negative inventory | 1 | 1 | `data.quantity < 0` check returns 400 (`inventory.py:357-358`) |
| Creates ADJUSTMENT movement with delta | 1 | 1 | Movement with correct delta (`inventory.py:364-374`) |
| Updates last_counted_at | 0.5 | 0.5 | Set to current time (`inventory.py:379`) |

**Subtotal: 18.5/19.5 → Normalized: 9.5/10**

**Feature Score: 9.5/10**

---

### F10: Dashboard Router (Weight: 5%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| GET /api/dashboard/ — 4 KPIs | 4 | 4 | total_inventory_value (1), pending_orders (1), low_stock_alerts (1), mtd_revenue (1) (`dashboard.py:14-56`) |
| GET /api/dashboard/order-pipeline — grouped by status | 3 | 3 | MongoDB aggregation with all 10 statuses, includes count and total_usd (`dashboard.py:59-78`) |
| GET /api/dashboard/inventory-health — per-SKU status | 3 | 3 | Per-SKU data with product_id, name, sku, quantity, reorder_point, status, value (`dashboard.py:81-115`) |

**Feature Score: 10/10**

---

### F11: Audit Service (Weight: 8%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `log_action` with correct signature | 1 | 1 | Has db, user_id, table_name, record_id, action, old_values, new_values, request (`services/audit.py:7`) |
| Logs CREATE with new_values | 1 | 1 | Called for all CRUD creates — products, suppliers, orders, payments |
| Logs UPDATE with old AND new values | 1.5 | 1.5 | Both passed in all update callers |
| Logs DELETE | 1 | 1 | Product soft delete calls log_action (`products.py:185`) |
| Excludes password_hash and mfa_secret | 1.5 | 1.5 | `EXCLUDED_FIELDS = {"password_hash", "mfa_secret", "_id"}` (`services/audit.py:4`) |
| Only logs when values change | 1.5 | 1.5 | Change detection: compares old vs new, skips if no changes (`services/audit.py:15-22`) |
| Captures IP from request.client.host | 0.5 | 0 | Code exists (`services/audit.py:26-27`) but **no caller ever passes `request` parameter** — always None |
| Captures User-Agent from headers | 0.5 | 0 | Same issue — `request` never passed by any caller |
| Audit integrated into all mutation endpoints | 1 | 0.5 | Products, suppliers, orders, payments have audit. **Inventory mutations (receive-shipment, sale, bulk-sale, adjust) do NOT call audit** |

**Feature Score: 8/10**

---

### F12: Inventory Monitor Service (Weight: 6%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `_calculate_days_until_stockout` | 2 | 0 | **Not implemented** — no `inventory_monitor.py` file exists |
| `daily_velocity = total_sold / 90` | 1 | 0 | **Not implemented** |
| Returns infinity when no sales | 1 | 0 | **Not implemented** |
| `_should_send_alert` — 24h dedup | 2 | 0 | **Not implemented** |
| `check_low_stock_and_alert` | 2 | 0 | **Not implemented** |
| Creates notification record | 1 | 0 | **Not implemented** |
| Integrates with email | 1 | 0 | **Not implemented** |

**Feature Score: 0/10**

---

### F13: Email Utility (Weight: 3%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `send_low_stock_alert` function | 2 | 0 | **Not implemented** — no `email.py` utility exists |
| HTML email with styled alert card | 3 | 0 | **Not implemented** |
| Includes SKU, qty, reorder_point, days | 2 | 0 | **Not implemented** |
| Action buttons | 1 | 0 | **Not implemented** |
| Sends via SMTP | 2 | 0 | **Not implemented** |

**Feature Score: 0/10**

---

### F14: Background Tasks / Scheduler (Weight: 3%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| Runs check_low_stock_and_alert periodically | 4 | 0 | **Not implemented** — no `scheduler.py` exists. Lifespan only does DB seeding |
| Integrated with FastAPI lifespan | 3 | 0 | **Not implemented** |
| Error handling | 2 | 0 | **Not implemented** |
| Graceful shutdown | 1 | 0 | **Not implemented** |

**Feature Score: 0/10**

---

### F15: Frontend Dashboard (Weight: 7%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| Client component ('use client') | 0.5 | 0 | Not Next.js — React CRA app (`frontend/src/App.js`) |
| Fetches GET /api/dashboard/kpis on mount | 1.5 | 0 | Fetches a generic `GET /api/` endpoint, not dashboard KPIs (`frontend/src/App.js:10-13`) |
| 4 KPI cards displayed | 2 | 0 | **No KPI cards.** Shows only "Building something incredible ~!" (`App.js:35`) |
| Responsive grid (1->2->4 columns) | 1 | 0 | **No grid layout** |
| Loading spinner state | 1 | 0 | **No loading state** |
| Error fallback with zeroes | 1 | 0 | **No error handling** |
| Order Pipeline section | 0.5 | 0 | **Not present** |
| Inventory Health section | 0.5 | 0 | **Not present** |
| TailwindCSS styling | 1 | 0 | Tailwind configured but App.js uses plain CSS classes (`App-header`, `App-link`) from `App.css` |
| Light theme | 0.5 | 0 | **Dark theme**: `background-color: #0f0f10` (`frontend/src/App.css:14`), white text |
| layout.tsx and globals.css | 0.5 | 0 | No `layout.tsx`. Has `index.css` (not `globals.css`) |

**Feature Score: 0/10**

---

### F16: Infrastructure — Docker & Nginx (Weight: 5%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| 3 services defined | 2 | 0 | **No `docker-compose.yml` file exists** |
| Frontend port + resource limits | 1 | 0 | **Not implemented** |
| Backend port + limits + health check | 1.5 | 0 | **Not implemented** |
| Nginx ports 80 + 443 + SSL | 1 | 0 | **Not implemented** |
| Custom bridge network | 0.5 | 0 | **Not implemented** |
| Backend env_file | 0.5 | 0 | **Not implemented** |
| Nginx proxies /api/ to backend | 1 | 0 | **Not implemented** |
| Nginx proxies / to frontend | 1 | 0 | **Not implemented** |
| SSL certs referenced | 0.5 | 0 | **Not implemented** |
| Valid YAML | 1 | 0 | **Not implemented** |

**Feature Score: 0/10**

---

### F17: Database Initialization Script (Weight: 4%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| Admin user: admin@example.com / admin123 bcrypt | 2 | 2 | Correct email, bcrypt rounds=12 (`services/seed.py:20-31`) |
| One supplier with contacts | 1.5 | 1.5 | "BabyCraft Toys Factory" with 2 contacts (Wang Li, Chen Wei) (`services/seed.py:33-59`) |
| 4-8 sample products | 1.5 | 1.5 | 8 products: LUMI-0003 through LUMI-2124 with full fields (`services/seed.py:62-99`) |
| 2 purchase orders (CONFIRMED + DRAFT) with line items | 2 | 2 | PO-2026-001 CONFIRMED with 3 items, PO-2026-002 DRAFT with 3 items (`services/seed.py:163-212`) |
| Inventory records (healthy + low stock) | 1.5 | 1.5 | 8 records: healthy (qty 50, 35, 22, 18), low (qty 8, 5, 3), out of stock (qty 0) (`services/seed.py:102-121`) |
| Stock movements (IN and OUT) | 1.5 | 1.5 | IN movements for 4 products, OUT movements for 2 products (`services/seed.py:123-148`) |

**Feature Score: 10/10**

---

### F18: Testing Suite (Weight: 12%)

#### F18a: Test Infrastructure

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| In-memory SQLite | 2 | 0 | **No test files exist at all** |
| Autouse fixtures | 1 | 0 | **No tests** |
| Override get_db dependency | 1.5 | 0 | **No tests** |
| pytest-asyncio | 0.5 | 0 | **No tests** |

#### F18b: Test Coverage

| Test File | Target | Max | Score | Evidence |
|-----------|--------|-----|-------|----------|
| test_products.py | 10 | 1 | 0 | **File does not exist** |
| test_orders.py | 9 | 1 | 0 | **File does not exist** |
| test_payments.py | 11 | 1 | 0 | **File does not exist** |
| test_inventory.py | 22 | 1.5 | 0 | **File does not exist** |
| test_suppliers.py | 10 | 1 | 0 | **File does not exist** |
| test_auth.py | 19 | 1.5 | 0 | **File does not exist** |
| test_audit.py | 18 | 1.5 | 0 | **File does not exist** |
| test_inventory_monitor.py | 18 | 1.5 | 0 | **File does not exist** |

#### F18c: Test Quality

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| Total test count >= 117 | 2 | 0 | 0 tests |
| All tests pass | 3 | 0 | N/A |
| Edge case coverage | 2 | 0 | N/A |
| Test independence | 1 | 0 | N/A |

**Subtotal: 0/22 → Normalized: 0/10**

**Feature Score: 0/10**

---

### F19: Security Implementation (Weight: 8%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| Passwords: bcrypt with work factor 12 | 1.5 | 1.5 | `bcrypt.gensalt(rounds=12)` (`auth.py:47`) |
| JWT: HS256 algorithm | 0.5 | 0.5 | `ALGORITHM = "HS256"` (`auth.py:13`) |
| JWT: 15-min access tokens | 0.5 | 0.5 | `ACCESS_TOKEN_EXPIRE_MINUTES = 15` (`auth.py:14`) |
| JWT: 7-day refresh tokens | 0.5 | 0.5 | `REFRESH_TOKEN_EXPIRE_DAYS = 7` (`auth.py:15`) |
| MFA: TOTP via pyotp | 1 | 0 | `pyotp` in `requirements.txt:89` but **never imported or used in any source file** |
| Rate limiting: 100/min global | 0.5 | 0 | `slowapi` in `requirements.txt:109` but **never imported or used** |
| Rate limiting: 5/min on auth | 0.5 | 0 | **Not implemented** |
| Input sanitization: bleach | 1 | 0 | `bleach` in `requirements.txt:9` but **never imported or used** |
| Encrypted storage: Fernet | 1.5 | 0 | **Not implemented.** `cryptography` in deps but Fernet never used |
| No raw SQL with user input | 1 | 1 | Uses MongoDB ORM (motor) throughout — no raw SQL anywhere |
| CORS: whitelist only | 0.5 | 0 | `CORS_ORIGINS="*"` in `backend/.env:3` |
| Security headers (4 total) | 1 | 0.75 | 3 of 4: X-Content-Type-Options, X-Frame-Options, Referrer-Policy (`server.py:68-70`). **Missing: Content-Security-Policy** |

**Feature Score: 4.75/10**

---

### F20: Business Logic Correctness (Weight: 7%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| Inventory Status Rules correct | 2 | 2 | `determine_status()`: OUT_OF_STOCK (qty==0), LOW (0<qty<=reorder_point), OK (qty>reorder_point) (`inventory.py:46-51`) |
| Days Until Stockout formula | 1.5 | 0 | **Not implemented** — no inventory_monitor service |
| Alert Deduplication — 24h | 1.5 | 0 | **Not implemented** |
| PO Number Generation — PO-YYYY-NNN | 1 | 1 | `generate_po_number()` with regex query and auto-increment (`orders.py:49-60`) |
| Payment ID Generation — PAY-YYYY-NNN | 1 | 1 | `generate_payment_id()` with same pattern (`payments.py:36-47`) |
| Atomic Bulk Operations | 2 | 2 | Both receive-shipment (`inventory.py:195-200`) and bulk-sale (`inventory.py:296-308`) pre-validate all items before modifying any |
| Database transactions — rollback on failure | 1 | 0.5 | MongoDB operations are not wrapped in transactions. Pre-validation approach mitigates but doesn't guarantee atomicity on failure mid-processing |

**Feature Score: 6.5/10**

---

## Final Score Card

| Feature | Weight | Score | Weighted |
|---------|--------|-------|----------|
| F01: Project Structure & Setup | 5% | 2.0 | 0.10 |
| F02: Database Schema & Models | 10% | 7.3 | 0.73 |
| F03: App Setup & Configuration | 5% | 3.0 | 0.15 |
| F04: Auth Router | 8% | 6.5 | 0.52 |
| F05: Products Router | 7% | 9.5 | 0.67 |
| F06: Suppliers Router | 5% | 10.0 | 0.50 |
| F07: Purchase Orders Router | 10% | 10.0 | 1.00 |
| F08: Payments Router | 7% | 8.5 | 0.60 |
| F09: Inventory Router | 15% | 9.5 | 1.43 |
| F10: Dashboard Router | 5% | 10.0 | 0.50 |
| F11: Audit Service | 8% | 8.0 | 0.64 |
| F12: Inventory Monitor Service | 6% | 0.0 | 0.00 |
| F13: Email Utility | 3% | 0.0 | 0.00 |
| F14: Background Tasks | 3% | 0.0 | 0.00 |
| F15: Frontend Dashboard | 7% | 0.0 | 0.00 |
| F16: Infrastructure (Docker/Nginx) | 5% | 0.0 | 0.00 |
| F17: DB Init Script | 4% | 10.0 | 0.40 |
| F18: Testing Suite | 12% | 0.0 | 0.00 |
| F19: Security Implementation | 8% | 4.75 | 0.38 |
| F20: Business Logic Correctness | 7% | 6.5 | 0.46 |
| **Weighted Total** | **140%** | | **8.08** |
| **Normalized (÷1.4)** | **100%** | | **5.77** |

**Final Weighted Average: 5.77/10 — Grade: C**

---

## Key Strengths
1. **Excellent core business logic** — Products, suppliers, orders, payments, and inventory endpoints are well-implemented with correct behavior
2. **Comprehensive seed data** — 8 products, realistic POs, varied inventory states
3. **Good audit trail** — Sensitive field exclusion, change detection, integrated into most mutations
4. **Correct PO workflow** — All 10 stages with forward-only enforcement

## Key Weaknesses
1. **Backend cannot start** — `server.py` imports from `routers/` but files are at root level
2. **Zero tests** — No test files whatsoever (12% weight)
3. **No infrastructure** — No Docker, no Nginx (5% weight)
4. **Placeholder frontend** — No dashboard, just "Building something incredible ~!"
5. **Missing services** — No inventory monitor, email utility, or scheduler (12% combined weight)
6. **Security gaps** — Libraries listed in requirements but never used (pyotp, slowapi, bleach, Fernet)
7. **Wrong technology stack** — MongoDB instead of SQLAlchemy/PostgreSQL, React CRA instead of Next.js 14
