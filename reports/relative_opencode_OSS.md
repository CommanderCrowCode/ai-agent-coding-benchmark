# Relative Grading Report: OpenCode OSS

## Summary
- **Weighted Average: 3.36/10**
- **Grade: F (Insufficient)**

> **Critical Issues:** The backend cannot start due to multiple import errors (`app/routers/__init__.py` imports `orders.py` which references non-existent `app/db` module). No database layer exists (`db.py`, `database.py` missing). Only 2 of 20+ required SQLAlchemy models are defined. Frontend has a JSX syntax error preventing compilation. No test infrastructure or test files (except one broken test). No `init_db.py`, no Alembic migrations, no `uv.lock`.

---

## Feature Scores

### F01: Project Structure & Setup (Weight: 5%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| Directory structure matches spec | /2 | 1 | Has `backend/`, `frontend/`, `infrastructure/` dirs with `routers/`, `schemas/`, `services/`, `utils/` subdirs. Missing `models.py` (split into separate files with no base), no `db.py` or `database.py` |
| `pyproject.toml` with all dependencies | /2 | 0.5 | Present (`backend/pyproject.toml`) but lists only fastapi, uvicorn, sqlalchemy, asyncpg, python-dotenv. Missing: jose, pyotp, passlib, slowapi, pydantic-settings, bleach, cryptography, alembic |
| `alembic.ini` + alembic directory | /1 | 0 | Completely missing |
| `uv` used for package management | /1 | 0 | No `uv.lock`, no evidence of uv usage |
| Frontend `package.json` with correct deps | /1 | 1 | Next.js 14.x, React 18.x, TailwindCSS 3.4 all present (`frontend/package.json`) |
| `tsconfig.json` and `tailwind.config.ts` | /1 | 1 | Both present and correctly configured |
| `.env` example or config documentation | /1 | 0 | No `.env.example` or config docs found |
| Clean separation of concerns | /1 | 0.5 | Has routers/schemas/services/utils structure but many routers use in-memory dicts instead of DB integration |

**Feature Score: 4/10**

---

### F02: Database Schema & Models (Weight: 10%)

#### F02a: Auth & Security Tables
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| `users` table | /2 | 0 | No users model. Auth uses in-memory `users_db = {}` dict (`routers/auth.py:16`) |
| `audit_log` table | /2 | 1 | `models/audit_log.py` exists but missing key fields: no `user_id`, no separate `old_values`/`new_values` (uses single `changes` JSON), no IP address, no user-agent |

#### F02b: Products & Suppliers Tables
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| `suppliers` — all fields | /1 | 0 | No SQLAlchemy model; router uses in-memory dict |
| `supplier_contacts` — FK, platform enum | /1 | 0 | No SQLAlchemy model |
| `products` — all fields | /2 | 0 | No SQLAlchemy model; router uses in-memory dict with simplified fields |
| `price_history` | /1 | 0 | No SQLAlchemy model; tracked in-memory only |

#### F02c: Purchase Orders Tables
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| `purchase_orders` | /2 | 0 | Referenced as `models.Order` in `routers/orders.py` but no model file exists |
| `po_line_items` | /1 | 0 | Referenced as `models.OrderLineItem` but no model file |
| `po_status_history` | /1 | 0 | Not implemented anywhere |

#### F02d: Shipments Tables
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| `shipments` | /1 | 0 | Missing entirely |
| `shipment_milestones` | /1 | 0 | Missing entirely |

#### F02e: Payments Table
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| `payments` | /2 | 0 | Referenced as `models.Payment` in `routers/payments.py` but no model file |

#### F02f: Inventory Tables
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| `inventory` | /2 | 0 | Missing |
| `stock_movements` | /1 | 0 | Missing |
| `sales` | /1 | 0 | Missing |

#### F02g: Planning Tables
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| `seasonality_factors` | /1 | 0 | Missing |
| `holidays` | /1 | 0 | Missing |
| `settings` | /0.5 | 0 | Missing |
| `notifications` | /0.5 | 0.5 | `models/notification.py` exists with `product_id` FK, but missing `alert_type` and `status` enum fields |

#### F02h: Indexes & Constraints
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| All specified indexes (11 total) | /3 | 0 | No indexes defined in any model |
| Unique constraints | /2 | 0 | No unique constraints defined |
| Foreign key relationships | /2 | 0 | Only `notification.product_id` FK exists, but `products` table doesn't exist |
| Proper use of enums | /1 | 0 | `PO_STATUS_ORDER` list in `orders.py` but not as SQLAlchemy Enum type |
| Table count >= 20 | /2 | 0 | Only 2 tables: `audit_logs` and `notifications` |

**Subtotal: 1.5/30, Normalized: 0.5/10**

---

### F03: Application Setup & Configuration (Weight: 5%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| FastAPI app with lifespan manager | /1 | 0 | Root `main.py` uses deprecated `@app.on_event("startup")`. `app/main.py` has no lifespan. |
| CORS configured for specified origins | /1 | 1 | Two origins whitelisted: `https://inventory.yourdomain.com` and `http://localhost:3000` (`app/main.py:24-27`) |
| Global rate limiter: 100 req/min | /1 | 1 | `slowapi` configured with `default_limits=["100/minute"]` (`app/main.py:15`) |
| Auth endpoint rate limit: 5 req/min | /1 | 0 | No auth-specific rate limit on `routers/auth.py` |
| Security headers middleware (all 4) | /2 | 1 | Has X-Content-Type-Options, X-Frame-Options, Referrer-Policy (3/4 correct). Has `Permissions-Policy` instead of `Content-Security-Policy` (`app/main.py:41-44`) |
| Health check `GET /health` | /1 | 1 | Present at `app/main.py:57-59`, returns `{"status": "ok"}` |
| Pydantic Settings with all env vars | /2 | 1.5 | `config.py` has DATABASE_URL, SECRET_KEY, DB_ENCRYPTION_KEY, ALGORITHM, token expiry, WISE keys, SMTP settings. Reasonably complete. |
| `database.py` — get_db dependency | /1 | 0 | **No `database.py` or `db.py` exists anywhere.** `routers/orders.py:8` and `routers/payments.py:8` import `from ..db import get_db` which would crash. |

**Feature Score: 5.5/10**

---

### F04: Auth Router (Weight: 8%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| POST /api/auth/register — bcrypt | /2 | 1.5 | Creates user with bcrypt hash via `CryptContext`. Uses in-memory `users_db` dict instead of DB. Default bcrypt rounds (not explicitly 12). (`auth.py:63-75`) |
| POST /api/auth/login — JWT tokens | /2 | 1.5 | Returns both access + refresh tokens via `Token` model. In-memory user lookup. (`auth.py:78-87`) |
| POST /api/auth/refresh | /1 | 1 | Decodes refresh token, generates new access + refresh tokens (`auth.py:90-103`) |
| POST /api/auth/mfa/setup — TOTP + QR | /1.5 | 1.5 | Generates TOTP secret via `pyotp.random_base32()`, returns provisioning URI (`auth.py:106-119`) |
| POST /api/auth/mfa/verify | /1.5 | 1.5 | Validates TOTP code via `totp.verify(code)`, enables MFA flag (`auth.py:122-131`) |
| JWT HS256, 15-min access | /1 | 1 | HS256 from `settings.ALGORITHM`, 15-min from `settings.ACCESS_TOKEN_EXPIRE_MINUTES` |
| 7-day refresh token | /0.5 | 0.5 | `settings.REFRESH_TOKEN_EXPIRE_DAYS = 7` |
| Rate limiting 5/min on auth | /0.5 | 0 | No auth-specific rate limiting applied |

**Feature Score: 7/10** *(Note: All auth logic uses in-memory storage, not a database. Code is functional but not production-ready.)*

---

### F05: Products Router (Weight: 7%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| GET /api/products — filtering | /2 | 1.5 | Both `best_seller` and `active` query params filter correctly. Uses in-memory dict. (`products.py:38-49`) |
| GET /api/products/{id} — with supplier | /1 | 0 | Returns product detail but does NOT include any supplier relationship data (`products.py:52-57`) |
| POST /api/products — create + price_history | /2 | 1.5 | Creates product and records initial price in `price_history_db`. In-memory only. (`products.py:60-67`) |
| PUT /api/products/{id} — price_history on change | /2 | 1.5 | Detects price change and appends to price history. (`products.py:70-83`) |
| DELETE soft delete | /1.5 | 1.5 | Sets `is_active = False` (`products.py:86-93`) |
| Proper Pydantic schemas | /1 | 0.5 | Schemas present but missing spec fields: `internal_sku`, `supplier_sku`, `age_min/max`, `units_per_carton`, `price_usd` |
| Input validation (SKU format) | /0.5 | 0 | No SKU format validation; no `internal_sku` field at all |

**Feature Score: 6/10** *(In-memory storage, missing many spec fields)*

---

### F06: Suppliers Router (Weight: 5%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| GET /api/suppliers — list | /1.5 | 1 | Works via in-memory dict (`suppliers.py:47-49`) |
| GET /api/suppliers/{id} — with contacts | /2 | 1 | Returns supplier detail but does NOT include contacts in response (`suppliers.py:52-57`) |
| GET /api/suppliers/{id}/contacts | /1.5 | 1 | Returns contacts filtered by supplier_id (`suppliers.py:60-69`) |
| POST /api/suppliers — create | /2 | 1 | Creates supplier in-memory, basic validation only (`suppliers.py:72-76`) |
| PUT /api/suppliers/{id} — update | /2 | 1 | Updates supplier in-memory (`suppliers.py:79-87`) |
| Proper Pydantic schemas | /1 | 0.5 | Present but missing spec fields: `legal_name`, `website`, `payment_terms`, `lead_time_days`, `moq`, `notes` |

**Feature Score: 5/10** *(In-memory storage, missing many spec fields, contacts not embedded in detail response)*

---

### F07: Purchase Orders Router (Weight: 10%)

> **CRITICAL BUG:** `routers/orders.py:8` imports `from ..db import get_db` but no `db.py` exists. This router **cannot be imported**, causing the entire backend to crash on startup.

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| GET /api/orders — filters | /1 | 0.5 | Code has status + supplier_id filters (`orders.py:40-53`) but cannot run |
| GET /api/orders/{id} — line_items + status_history | /1.5 | 0.5 | Schema `OrderDetailRead` defines `line_items` and `status_history` fields but no eager loading |
| GET /api/orders/workflow-stats | /1 | 0.5 | Group-by query code exists (`orders.py:65-72`) |
| POST /api/orders — with line items | /1 | 0.5 | Creates order + line items (`orders.py:75-105`) |
| Auto-generate PO-YYYY-NNN | /1 | 0.5 | Correct format with auto-increment (`orders.py:84`) |
| Calculate totals from line items | /1 | 0 | **No auto-calculation of subtotal from line items** |
| PUT /api/orders/{id} — update | /0.5 | 0.25 | Updates notes and expected_ready_date (`orders.py:108-123`) |
| POST /api/orders/{id}/status — advance | /1 | 0.5 | Status advancement with validation (`orders.py:126-137`) |
| All 10 stages defined | /1 | 0.5 | All 10 stages in `PO_STATUS_ORDER` list (`orders.py:13-24`) but never tested |
| Forward-only transitions enforced | /1 | 0.5 | `_validate_status_transition` enforces `new_idx == current_idx + 1` (`orders.py:27-37`) |
| Status change recorded in history | /0.5 | 0 | **No `po_status_history` recording** |

**Feature Score: 3.5/10** *(Code architecture is reasonable but fundamentally non-functional due to missing `db.py` dependency)*

---

### F08: Payments Router (Weight: 7%)

> **CRITICAL BUG:** Same as F07 — `routers/payments.py:8` imports `from ..db import get_db` which doesn't exist.

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| GET /api/payments — filter | /1 | 0.5 | Status filter code exists (`payments.py:13-18`) |
| GET /api/payments/{id} — detail | /1 | 0.5 | Code exists (`payments.py:21-26`) |
| GET /api/payments/summary | /2 | 1 | Monthly total, YTD total, total fees all calculated (`payments.py:29-48`) |
| POST /api/payments — PAY-YYYY-NNN | /2 | 1 | Correct `PAY-YYYY-NNN` format with auto-increment (`payments.py:54-72`) |
| PUT /api/payments/{id} — update + audit | /2 | 1 | Updates status/notes and calls `log_action` (`payments.py:75-94`) |
| GET /api/payments/{id}/sync-wise stub | /1 | 0.5 | Stub returns `{"status": "stub", "payment_id": id}` (`payments.py:97-100`) |
| Proper Pydantic schemas | /1 | 0.5 | Present in `schemas/payment.py` but simplified (missing supplier_id, Wise fields, payment_id field) |

**Feature Score: 3.5/10** *(Non-functional due to missing `db.py`; code logic is reasonable)*

---

### F09: Inventory Router (Weight: 15%)

> **ALL endpoints return HTTP 501 "Not implemented"** (`routers/inventory.py:33,38,43,48,53,58,63,68`)

#### F09a: Read Endpoints
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| GET /api/inventory/ — list with status | /2 | 0 | Returns 501 |
| Status filter query param | /1 | 0 | Returns 501 |
| Uses joinedload | /0.5 | 0 | Returns 501 |
| GET /api/inventory/stats | /1.5 | 0 | Returns 501 |
| GET /api/inventory/{product_id} — auto-create | /1 | 0 | Returns 501 |
| GET /api/inventory/{product_id}/movements | /1 | 0 | Returns 501 |

#### F09b: Receive Shipment
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| POST /api/inventory/receive-shipment | /1 | 0 | Returns 501 |
| Validates all products exist | /1.5 | 0 | Returns 501 |
| Creates IN stock movement | /1 | 0 | Returns 501 |
| Updates inventory quantity | /0.5 | 0 | Returns 501 |

#### F09c: Sales
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| POST /api/inventory/sale | /1 | 0 | Returns 501 |
| Creates OUT movement + sale record | /1 | 0 | Returns 501 |
| Decrements inventory | /0.5 | 0 | Returns 501 |
| POST /api/inventory/bulk-sale | /1.5 | 0 | Returns 501 |
| Insufficient stock returns 400 | /1 | 0 | Returns 501 |

#### F09d: Adjustment
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| POST /api/inventory/adjust | /1 | 0 | Returns 501 |
| Prevents negative inventory | /1 | 0 | Returns 501 |
| Creates ADJUSTMENT movement | /1 | 0 | Returns 501 |
| Updates last_counted_at | /0.5 | 0 | Returns 501 |

**Subtotal: 0/18, Normalized: 0/10** *(Pydantic schemas exist in `schemas/inventory.py` showing awareness of the interface, but zero implementation)*

**Feature Score: 1/10** *(Route stubs and schemas exist, giving minimal credit)*

---

### F10: Dashboard Router (Weight: 5%)

> **ALL endpoints return HTTP 501 "Not implemented"** (`routers/dashboard.py:8,13,18`)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| GET /api/dashboard/ — 4 KPIs | /4 | 0 | Returns 501 |
| GET /api/dashboard/order-pipeline | /3 | 0 | Returns 501 |
| GET /api/dashboard/inventory-health | /3 | 0 | Returns 501 |

**Feature Score: 0.5/10** *(Route stubs exist)*

---

### F11: Audit Service (Weight: 8%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| `log_action` function with correct signature | /1 | 0.5 | Function exists but signature is `(db, action, model_name, record_id, new_data)` — missing `user_id`, `table_name`/`old_values`/`new_values` as separate params, missing `request` (`services/audit.py:22-24`) |
| Logs CREATE with new_values | /1 | 0.5 | Handles creation case, logs non-sensitive fields as `{key: {old: None, new: val}}` (`audit.py:34-39`) |
| Logs UPDATE with old AND new | /1.5 | 1 | `_filter_changes` compares old record attributes with new data (`audit.py:11-19`) |
| Logs DELETE actions | /1 | 0 | No DELETE handling in the service |
| Excludes password_hash and mfa_secret | /1.5 | 1.5 | `SENSITIVE_FIELDS = {"password_hash", "mfa_secret"}` with explicit filtering (`audit.py:8`) |
| Only logs when values change | /1.5 | 1.5 | Returns early with `if not changes: return` (`audit.py:40-41`) |
| Captures IP address | /0.5 | 0 | No `request` parameter, no IP capture |
| Captures User-Agent | /0.5 | 0 | No user-agent capture |
| Integrated into all mutation endpoints | /1 | 0.5 | Only called from `payments.py:91` update endpoint. Not in products, suppliers, orders, or auth. |

**Feature Score: 5/10**

---

### F12: Inventory Monitor Service (Weight: 6%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| `_calculate_days_until_stockout` — 90-day sales query | /2 | 0.5 | Function exists but does NOT query last 90 days of sales. Uses hardcoded `sales_velocity=1.0` default (`services/inventory_monitor.py:10-18`) |
| `daily_velocity = total_sold / 90` formula | /1 | 0 | Formula is `product.quantity_on_hand / sales_velocity` with default velocity=1.0, not calculated from actual sales data |
| Returns infinity when no sales | /1 | 1 | Returns `float("inf")` when `sales_velocity <= 0` (`inventory_monitor.py:17`) |
| `_should_send_alert` — 24h dedup | /2 | 1 | In-memory `_last_alert_sent` dict checks 24h window (`inventory_monitor.py:21-25`). Does NOT check notifications table. |
| `check_low_stock_and_alert` — iterates LOW/OUT products | /2 | 1 | Queries products with `quantity_on_hand < 10` — hardcoded threshold, not using `reorder_point` (`inventory_monitor.py:30-31`) |
| Creates notification record | /1 | 1 | Creates `Notification` object and commits (`inventory_monitor.py:38-44`) |
| Integrates with email | /1 | 0.5 | Calls `send_low_stock_alert` but email utility just uses `print()` (`utils/email.py:19`) |

**Feature Score: 4.5/10** *(References `models.Product.quantity_on_hand` which doesn't exist as a model, so would crash at runtime)*

---

### F13: Email Utility (Weight: 3%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| `send_low_stock_alert` with correct params | /2 | 1 | Function exists but params are `(product, days_until_stockout)` — missing `sku`, `qty`, `reorder_point`, `recipients` (`utils/email.py:4`) |
| HTML email with styled alert card | /3 | 1 | Basic HTML template with `<h2>`, `<p>` tags and inline style. Not a proper styled card. (`email.py:7-17`) |
| Includes SKU, qty, reorder_point, days | /2 | 1 | Has product name, ID, quantity, days. Missing SKU and reorder_point. |
| Action buttons (Create PO, View Inventory) | /1 | 0 | No action buttons in the HTML |
| Sends via SMTP | /2 | 0 | **Uses `print()` instead of SMTP.** No email actually sent. (`email.py:19`) |

**Feature Score: 3/10**

---

### F14: Background Tasks / Scheduler (Weight: 3%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| Runs check_low_stock_and_alert periodically | /4 | 1 | `scheduler_loop` has `asyncio.sleep(3600)` (hourly). But `_run_inventory_monitor` passes `products = []` (empty list TODO comment) and doesn't pass a db session. (`tasks/scheduler.py:6-15`) |
| Integrated with FastAPI lifespan | /3 | 1 | Root `main.py` uses deprecated `@app.on_event("startup")` with `asyncio.create_task(scheduler_loop())`. Not integrated in `app/main.py` at all. |
| Error handling | /2 | 0 | No try/catch anywhere in scheduler |
| Graceful shutdown | /1 | 0 | No task cancellation on shutdown |

**Feature Score: 2/10**

---

### F15: Frontend Dashboard (Weight: 7%)

> **CRITICAL BUG:** JSX syntax error on `page.tsx:59` — extra `)` in `className={...`)}>`. Also, dynamic Tailwind classes like `bg-${card.color}-100` won't work (Tailwind can't detect dynamic classes during purge). **Missing `layout.tsx`.**

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| Client component (`'use client'`) | /0.5 | 0.5 | Present at `page.tsx:1` |
| Fetches GET /api/dashboard/ on mount | /1.5 | 0.5 | `useEffect` with `fetch('/api/dashboard/')` code exists (`page.tsx:24-38`) but component won't compile |
| 4 KPI cards displayed | /2 | 0.5 | All 4 cards defined: Inventory Value, Pending Orders, Low Stock, MTD Revenue (`page.tsx:40-45`) but won't render |
| Responsive grid (1->2->4 columns) | /1 | 0.5 | `grid-cols-1 sm:grid-cols-2 lg:grid-cols-4` (`page.tsx:57`) but won't render |
| Loading spinner state | /1 | 0.5 | SVG spinner in loading state (`page.tsx:49-55`) but won't render |
| Error fallback with zeroes | /1 | 0.5 | Catches error, defaults to `defaultData` with zeroes (`page.tsx:34-36`) but won't render |
| Order Pipeline section | /0.5 | 0.25 | Placeholder section exists (`page.tsx:67-69`) |
| Inventory Health section | /0.5 | 0.25 | Placeholder section exists (`page.tsx:71-73`) |
| TailwindCSS styling throughout | /1 | 0.5 | Tailwind classes used but dynamic classes `bg-${color}-100` won't be purged |
| Light theme | /0.5 | 0.25 | White background intent, correct color palette in `tailwind.config.ts:12-16` |
| `layout.tsx` and `globals.css` present | /0.5 | 0.25 | `globals.css` has Tailwind directives. **`layout.tsx` is MISSING.** |

**Feature Score: 4/10** *(Good design intent but syntax error prevents compilation; missing layout.tsx)*

---

### F16: Infrastructure — Docker & Nginx (Weight: 5%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| 3 services defined | /2 | 2 | `frontend`, `backend`, `nginx` all defined (`docker-compose.yml:3,18,42`) |
| Frontend: port 3000, limits (0.5 CPU / 512MB) | /1 | 1 | Port 3000, cpus: '0.5', memory: 512M (`docker-compose.yml:10-16`) |
| Backend: port 8000, limits, healthcheck 30s | /1.5 | 1.5 | Port 8000, cpus: '0.75', memory: 1G, interval: 30s (`docker-compose.yml:28-39`) |
| Nginx: ports 80+443, SSL | /1 | 1 | Both ports exposed, SSL with cert paths (`docker-compose.yml:45-46`, `nginx.conf:26-30`) |
| Custom bridge network | /0.5 | 0.5 | `appnet` with `driver: bridge` (`docker-compose.yml:57-58`) |
| Backend reads from .env | /0.5 | 0.5 | `env_file: - .env` (`docker-compose.yml:25-26`) |
| Nginx proxies /api/ to backend:8000 | /1 | 1 | `proxy_pass http://backend/` for `/api/` location (`nginx.conf:32-35`) |
| Nginx proxies / to frontend:3000 | /1 | 1 | `proxy_pass http://frontend/` for `/` location (`nginx.conf:37-40`) |
| SSL certs in ./certs/ | /0.5 | 0.5 | `./certs:/etc/nginx/certs:ro` (`docker-compose.yml:50`) |
| Valid and well-structured | /1 | 1 | Valid YAML, clean structure, proper `depends_on` |

**Feature Score: 10/10**

---

### F17: Database Initialization Script (Weight: 4%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| Admin user with bcrypt hash | /2 | 0 | **No `init_db.py` file exists anywhere** |
| Supplier with contacts | /1.5 | 0 | Missing |
| 4-8 sample products | /1.5 | 0 | Missing |
| 2 purchase orders with line items | /2 | 0 | Missing |
| Inventory records | /1.5 | 0 | Missing |
| Sample stock movements | /1.5 | 0 | Missing |

**Feature Score: 0/10**

---

### F18: Testing Suite (Weight: 12%)

#### F18a: Test Infrastructure
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| In-memory SQLite for test isolation | /2 | 0 | No `conftest.py`, no test database configuration |
| Autouse fixtures for setup/teardown | /1 | 0 | No fixtures defined |
| Override `get_db()` dependency | /1.5 | 0 | No dependency override |
| pytest-asyncio configured | /0.5 | 0 | Listed in pyproject.toml optional deps but not configured |

#### F18b: Test Coverage by Module

| Test File | Target Count | Points | Score | Evidence |
|-----------|-------------|--------|-------|---------|
| `test_products.py` | 10 | /1 | 0 | Missing |
| `test_orders.py` | 9 | /1 | 0 | Missing |
| `test_payments.py` | 11 | /1 | 0 | Missing |
| `test_inventory.py` | 22 | /1.5 | 0 | Missing |
| `test_suppliers.py` | 10 | /1 | 0 | Missing |
| `test_auth.py` | 19 | /1.5 | 0 | Missing |
| `test_audit.py` | 18 | /1.5 | 0 | Missing |
| `test_inventory_monitor.py` | 18 | /1.5 | 0 | Missing |

Only file present: `test_health.py` (1 test, references undefined `client` fixture)

#### F18c: Test Quality
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| Total tests >= 117 | /2 | 0 | 1 test total |
| All tests pass | /3 | 0 | `test_health.py` references `client` fixture that doesn't exist (no conftest.py) — would fail |
| Edge case coverage | /2 | 0 | No edge cases |
| Tests independent | /1 | 0 | Only 1 test |

**Subtotal: 0/22, Normalized: 0/10**

---

### F19: Security Implementation (Weight: 8%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| Passwords: bcrypt with work factor 12 | /1.5 | 0.5 | `CryptContext(schemes=["bcrypt"], deprecated="auto")` — uses bcrypt but default rounds, not explicitly 12 (`auth.py:13`) |
| JWT: HS256 | /0.5 | 0.5 | `ALGORITHM: str = "HS256"` (`config.py:8`) |
| JWT: 15-min access tokens | /0.5 | 0.5 | `ACCESS_TOKEN_EXPIRE_MINUTES: int = 15` (`config.py:9`) |
| JWT: 7-day refresh tokens | /0.5 | 0.5 | `REFRESH_TOKEN_EXPIRE_DAYS: int = 7` (`config.py:10`) |
| MFA: TOTP via pyotp | /1 | 1 | `import pyotp` with `random_base32()` and `TOTP.verify()` (`auth.py:6,113,127-128`) |
| Rate limiting: 100/min global | /0.5 | 0.5 | `Limiter(key_func=get_remote_address, default_limits=["100/minute"])` (`main.py:15`) |
| Rate limiting: 5/min on auth | /0.5 | 0 | No auth-specific rate limit |
| Input sanitization: bleach | /1 | 0 | No bleach import or usage anywhere |
| Encrypted storage: Fernet | /1.5 | 0 | `DB_ENCRYPTION_KEY` in config but Fernet never imported or used |
| No raw SQL with user input | /1 | 1 | ORM used throughout — no `text()` calls in routers |
| CORS: whitelist only | /0.5 | 0.5 | Specific origins listed (`main.py:24-27`) |
| Security headers (4) | /1 | 0.5 | 3 of 4 correct: X-Content-Type-Options, X-Frame-Options, Referrer-Policy. Has Permissions-Policy instead of CSP. (`main.py:41-44`) |

**Feature Score: 5/10**

---

### F20: Business Logic Correctness (Weight: 7%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|---------|
| Inventory Status Rules correct | /2 | 0 | Not implemented — inventory router returns 501 |
| Days Until Stockout formula | /1.5 | 0 | Uses hardcoded velocity=1.0, not 90-day sales calculation (`inventory_monitor.py:14`) |
| Alert Deduplication — 24h | /1.5 | 1 | In-memory `_last_alert_sent` with 24h check (`inventory_monitor.py:21-25`) |
| PO Number Generation — PO-YYYY-NNN | /1 | 0.5 | Format correct in code (`orders.py:84`) but router can't be imported |
| Payment ID Generation — PAY-YYYY-NNN | /1 | 0.5 | Format correct in code (`payments.py:61`) but router can't be imported |
| Atomic Bulk Operations | /2 | 0 | Not implemented — inventory router returns 501 |
| Database transactions — rollback | /1 | 0.5 | `db.commit()` used in orders/payments code but no explicit rollback handling |

**Feature Score: 2.5/10**

---

## Final Score Card

| Feature | Weight | Score |
|---------|--------|-------|
| F01: Project Structure & Setup | 5% | 4.0 |
| F02: Database Schema & Models | 10% | 0.5 |
| F03: App Setup & Configuration | 5% | 5.5 |
| F04: Auth Router | 8% | 7.0 |
| F05: Products Router | 7% | 6.0 |
| F06: Suppliers Router | 5% | 5.0 |
| F07: Purchase Orders Router | 10% | 3.5 |
| F08: Payments Router | 7% | 3.5 |
| F09: Inventory Router | 15% | 1.0 |
| F10: Dashboard Router | 5% | 0.5 |
| F11: Audit Service | 8% | 5.0 |
| F12: Inventory Monitor Service | 6% | 4.5 |
| F13: Email Utility | 3% | 3.0 |
| F14: Background Tasks | 3% | 2.0 |
| F15: Frontend Dashboard | 7% | 4.0 |
| F16: Infrastructure (Docker/Nginx) | 5% | 10.0 |
| F17: DB Init Script | 4% | 0.0 |
| F18: Testing Suite | 12% | 0.0 |
| F19: Security Implementation | 8% | 5.0 |
| F20: Business Logic Correctness | 7% | 2.5 |
| **Weighted Total** | **140%** | **4.70 raw** |
| **Normalized Weighted Average** | **100%** | **3.36/10** |

*Note: Rubric weights sum to 140%. Weighted average normalized as: 4.70 / 1.40 = 3.36/10*

**Final Grade: F (Insufficient)**

---

## Key Findings

### What Works
- Infrastructure (Docker Compose + Nginx) is production-quality
- Auth router has solid JWT + MFA implementation (in-memory)
- Products and suppliers routers have functional CRUD (in-memory)
- Security headers, CORS, global rate limiting configured
- Frontend code design shows strong React/Tailwind knowledge

### What's Fundamentally Broken
- **No database layer** — `db.py`/`database.py` is missing, causing import crashes
- **Only 2 of 20+ models** — audit_log and notification tables only
- **Backend cannot start** — import chain crashes on `from ..db import get_db`
- **Frontend won't compile** — JSX syntax error on `page.tsx:59`
- **Zero test infrastructure** — no conftest.py, no test DB, 1 broken test
- **Inventory & Dashboard completely unimplemented** — all 501 stubs
- **No init_db.py** — no seed data
- **No Alembic migrations** — no schema management
