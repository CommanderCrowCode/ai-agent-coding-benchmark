# Relative Grading Report: OpenCode Nemotron

## Summary
- Weighted Average: 4.26/10
- Grade: D

## Feature Scores

### F01: Project Structure & Setup (Weight: 5%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| Directory structure matches spec | /2 | 1 | Has `backend/` with `app/`, `tests/`, and `frontend/`, `infrastructure/` dirs. Missing `app/utils/`, `app/tasks/`, `app/routers/dashboard.py`. Close but missing dirs. |
| `pyproject.toml` with all dependencies | /2 | 1 | `pyproject.toml` exists (`backend/pyproject.toml:1-32`) but missing critical deps: `passlib`, `pyotp`, `bcrypt`, `slowapi`, `pydantic-settings`, `bleach`, `cryptography`. Only has fastapi, uvicorn, sqlalchemy, psycopg2-binary, pydantic, python-dotenv, jinja2, httpx. |
| `alembic.ini` + alembic directory exists | /1 | 0.5 | `alembic.ini` exists (`backend/alembic.ini:1-12`) but no `alembic/` directory with migration scripts. |
| `uv` used for package management | /1 | 1 | `uv.lock` file present at `backend/uv.lock`. |
| Frontend `package.json` with correct deps | /1 | 1 | `frontend/package.json:11-16` has Next.js >=14, React >=18, TailwindCSS ^3.4.0. |
| `tsconfig.json` and `tailwind.config.ts` | /1 | 1 | Both present: `frontend/tsconfig.json`, `frontend/tailwind.config.ts`. |
| `.env` example or config documentation | /1 | 0.5 | `infrastructure/.env` exists but is minimal (only DATABASE_URL). Not a proper example file. |
| Clean separation of concerns | /1 | 0.5 | Has `routers/`, `schemas/`, `services/` dirs but no `__init__.py` files anywhere, making packages non-functional. Missing `utils/`, `tasks/`. |

**Feature Score: 6.5/10**

---

### F02: Database Schema & Models (Weight: 10%)

#### F02a: Auth & Security Tables
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `users` table — all fields | /2 | 2 | `models.py:83-99`: email, password_hash, mfa_secret, mfa_enabled, role, is_active all present. |
| `audit_log` table — all fields including JSON old/new values | /2 | 2 | `models.py:102-120`: table_name, record_id, action, old_values (JSON), new_values (JSON), user_id, ip_address, user_agent, created_at. |

#### F02b: Products & Suppliers Tables
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `suppliers` — all fields | /1 | 1 | `models.py:126-141`: name, legal_name, website, address, payment_terms, lead_time_days, moq, notes. Complete. |
| `supplier_contacts` — FK to supplier, platform enum | /1 | 1 | `models.py:144-158`: supplier_id FK, platform field present. |
| `products` — all fields with constraints | /2 | 1 | `models.py:161-183`: Has internal_sku (unique), supplier_sku, name, age_range_start/end, price_usd, units_per_carton, is_best_seller, is_active, supplier FK. But MISSING primary key `id` column — critical structural defect. |
| `price_history` — correct fields | /1 | 1 | `models.py:185-198`: product_id FK, supplier_id FK, price_usd, effective_date. |

#### F02c: Purchase Orders Tables
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `purchase_orders` — all fields with constraints | /2 | 1 | `models.py:204-224`: po_number (unique), status, financial fields (subtotal_usd, shipping_usd, other_fees_usd, total_usd). But MISSING primary key `id` column. |
| `po_line_items` — FK to po and product | /1 | 1 | `models.py:226-236`: po_id FK, product_id FK, quantity, unit_price_usd. |
| `po_status_history` — correct fields | /1 | 1 | `models.py:239-251`: po_id FK, old_status, new_status, changed_by, changed_at. |

#### F02d: Shipments Tables
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `shipments` — complete | /1 | 1 | `models.py:257-270`: po_id FK, carrier, tracking_number, bill_of_lading, origin_port, destination_port, status. |
| `shipment_milestones` — correct with enum | /1 | 0.5 | `models.py:273-287`: shipment_id FK, milestone_type (String, not enum), expected/actual dates. Enum type defined but stored as String. |

#### F02e: Payments Table
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `payments` — all fields | /2 | 2 | `models.py:293-313`: payment_id (PAY-YYYY-NNN unique), po_id FK, supplier_id FK, wise_transfer_id, wise_tracking_id, status, amount_usd, fee_usd, amount_received, currency_received, notes. |

#### F02f: Inventory Tables
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `inventory` — all fields with defaults | /2 | 1.5 | `models.py:330-345`: product_id (unique FK), quantity (default=0), reorder_point (default=10), safety_stock_days (default=14), last_counted_at. Missing primary key `id`. |
| `stock_movements` — correct with enums | /1 | 1 | `models.py:348-361`: movement_type (IN/OUT/ADJUSTMENT enum defined), reference_type, reference_id. |
| `sales` — correct fields | /1 | 1 | `models.py:364-378`: product_id FK, platform (SHOPEE/LAZADA/DIRECT), order_reference, sale_date, quantity, unit_price_usd. |

#### F02g: Planning Tables
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `seasonality_factors` | /1 | 1 | `models.py:385-389`: month (1-12 primary key), multiplier. |
| `holidays` | /1 | 1 | `models.py:392-400`: country, holiday_name, start_date, end_date, type. |
| `settings` | /0.5 | 0.5 | `models.py:403-407`: key/value store. |
| `notifications` | /0.5 | 0.5 | `models.py:410-419`: product_id FK, alert_type, message, status, created_at. |

#### F02h: Indexes & Constraints
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| All specified indexes created (11 total) | /3 | 1 | 11 unique index names declared, BUT many are on the WRONG tables. E.g., `idx_audit_log_*` indexes on User table (`models.py:86-88`), `idx_purchase_orders_*` on SupplierContact (`models.py:147-149`), `idx_price_history_product_date` on Product (`models.py:165`) referencing non-existent columns. These would crash at table creation. ~4-5 correctly placed. |
| Unique constraints on internal_sku, po_number, payment_id | /2 | 2 | All three present: `models.py:168` (internal_sku unique=True), `models.py:211` (po_number unique=True), `models.py:300` (payment_id unique=True). |
| Foreign key relationships correctly defined | /2 | 1.5 | Most FKs correct, but some relationship `back_populates` are inconsistent (e.g., Inventory has `stock_movements` and `sales` that conflict with Product's same-named relationships). |
| Proper use of enums for status fields | /1 | 0.5 | Enums defined as Python classes (`models.py:28-78`) but stored as String columns, not DB-level enums. |
| Table count >= 20 | /2 | 2 | Exactly 20 model classes inheriting from Base (User, AuditLog, Supplier, SupplierContact, Product, PriceHistory, PurchaseOrder, POLineItem, POStatusHistory, Shipment, ShipmentMilestone, Payment, OrderItem, Inventory, StockMovement, Sale, SeasonalityFactor, Holiday, Settings, Notification). |

**Subtotal: 24/30 → Normalized: 8.0/10**

**Feature Score: 8.0/10**

---

### F03: Application Setup & Configuration (Weight: 5%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| FastAPI app with lifespan manager | /1 | 0.5 | `main.py:21-27`: Lifespan defined but only calls `await init_db()` which is a no-op pass statement (`database.py:16-19`). No background task scheduling. |
| CORS configured for specified origins | /1 | 1 | `main.py:35-41`: Both `https://inventory.yourdomain.com` and `http://localhost:3000` whitelisted. |
| Global rate limiter: 100 req/min | /1 | 0.5 | `main.py:43-46`: Limiter created with 100/min, but `rate_limiter_middleware` is not a proper middleware class — `app.add_middleware()` would crash. |
| Auth endpoint rate limit: 5 req/min | /1 | 0 | `main.py:48-49`: `auth_rate_limiter_middleware` is also not a proper middleware class. Would crash. Not functional. |
| Security headers middleware (all 4 headers) | /2 | 0 | `security.py:36-42`: Middleware is a plain function, not a class — `app.add_middleware()` would crash. Even if it worked: has X-Content-Type-Options, X-Frame-Options, Referrer-Policy (3 of 4). Missing CSP. Has X-XSS-Protection instead. |
| Health check `GET /health` returns `{"status": "ok"}` | /1 | 0.5 | `health.py:6-8`: Endpoint exists at `/health`, returns correct response. But registered as `prefix="/api"` in main.py so actual path is `/api/health`, not `/health`. |
| Pydantic Settings with all env vars | /2 | 2 | `config.py:1-27`: All env vars listed: DATABASE_URL, DB_ENCRYPTION_KEY, SECRET_KEY, ALGORITHM, ACCESS_TOKEN_EXPIRE_MINUTES, REFRESH_TOKEN_EXPIRE_DAYS, WISE_API_KEY, WISE_WEBHOOK_SECRET, SMTP_*, FROM_EMAIL, DEBUG. |
| `app/database.py` — proper session management, `get_db` dependency | /1 | 0 | `database.py:1-19`: No `Base` defined, no `get_db` dependency, no `SessionLocal`. Routers import `get_db` from here but it doesn't exist. Fundamentally broken. |

**Feature Score: 4.5/10**

---

### F04: Auth Router (Weight: 8%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `POST /api/auth/register` — creates user, hashes password with bcrypt | /2 | 1 | `routers/auth.py:69-84`: Endpoint exists, uses `CryptContext(schemes=["bcrypt"])` but default work factor (not 12). Would fail at runtime due to missing `get_db` import. |
| `POST /api/auth/login` — returns JWT access + refresh tokens | /2 | 0 | `routers/auth.py:87-94`: Returns `{"message": "Login successful", "user_id": user.id}` — NO JWT tokens generated. Comment says "TODO: generate JWT tokens". |
| `POST /api/auth/refresh` — exchanges refresh token | /1 | 0 | `routers/auth.py:97-101`: Stub that returns `{"message": "Refresh token received"}`. TODO comment. |
| `POST /api/auth/mfa/setup` — generates TOTP secret, returns QR URI | /1.5 | 1.5 | `routers/auth.py:104-119`: Uses `pyotp.random_base32()`, returns mfa_secret and `provisioning_uri()` QR URI. Logic is correct. |
| `POST /api/auth/mfa/verify` — validates TOTP code, enables MFA | /1.5 | 1.5 | `routers/auth.py:122-133`: Uses `pyotp.TOTP(user.mfa_secret).verify(request.token)`. Validates and enables MFA. |
| JWT uses HS256 with 15-min access token expiry | /1 | 0 | No JWT implementation at all. Config has `ALGORITHM` and `ACCESS_TOKEN_EXPIRE_MINUTES` but no JWT encoding/decoding code. |
| Refresh token: 7-day expiry | /0.5 | 0 | No refresh token implementation. |
| Rate limiting: 5 req/min on auth endpoints | /0.5 | 0 | Middleware is broken (not a proper class). No functional rate limiting. |

**Feature Score: 4.0/10**

---

### F05: Products Router (Weight: 7%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `GET /api/products` — list with filtering (best_seller, active) | /2 | 1.5 | `routers/products.py:14-26`: Both filters present via `status_filter` param. Uses string comparison. Would fail at runtime (missing `get_db`). |
| `GET /api/products/{id}` — detail with supplier info | /1 | 0.5 | `routers/products.py:29-38`: Returns product but does NOT eagerly load or include supplier relationship data. |
| `POST /api/products` — create product, record initial price in price_history | /2 | 0.5 | `routers/products.py:41-56`: Creates product but does NOT record initial price in price_history. Also has bug: `Product(**request.dict(), supplier_id=request.supplier_id)` duplicates supplier_id. |
| `PUT /api/products/{id}` — update; records price_history if price changed | /2 | 0.5 | `routers/products.py:59-76`: Updates fields but does NOT detect price changes or record price_history. |
| `DELETE /api/products/{id}` — soft delete (is_active=False) | /1.5 | 1.5 | `routers/products.py:79-91`: Correctly sets `is_active = False`. |
| Proper Pydantic schemas for request/response | /1 | 1 | `schemas/products.py:1-43`: Well-defined ProductBase, ProductCreate, ProductUpdate, ProductOut with orm_mode. |
| Input validation (e.g., valid SKU format) | /0.5 | 0.5 | `schemas/products.py:9-10`: SKU pattern validation `^LUMI-\d{4}$`. |

**Feature Score: 6.0/10**

---

### F06: Suppliers Router (Weight: 5%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `GET /api/suppliers` — list all | /1.5 | 0 | `routers/suppliers.py:18-24`: Endpoint exists but imports are broken (imports `Contact` instead of `SupplierContact`, `from ..schemas import` without `__init__.py`, missing `select` import, redefines `get_db` incorrectly). Would crash. |
| `GET /api/suppliers/{id}` — detail with contacts | /2 | 0 | `routers/suppliers.py:28-37`: Exists but non-functional due to broken imports. |
| `GET /api/suppliers/{id}/contacts` — contacts only | /1.5 | 0 | `routers/suppliers.py:41-47`: Exists but non-functional. |
| `POST /api/suppliers` — create | /2 | 0 | `routers/suppliers.py:52-61`: Exists but has bug: `.dict` instead of `.dict()` (missing parens, line 57). Non-functional. |
| `PUT /api/suppliers/{id}` — update | /2 | 0 | `routers/suppliers.py:65-79`: Exists but non-functional due to broken imports. |
| Proper Pydantic schemas | /1 | 1 | `schemas/suppliers.py:1-37`: SupplierBase, SupplierCreate, SupplierUpdate, SupplierOut with orm_mode. Well-defined. |

**Feature Score: 1.0/10**

---

### F07: Purchase Orders Router (Weight: 10%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `GET /api/orders` — list with filters (status, supplier_id) | /1 | 0.5 | `routers/orders.py:80-93`: Both filters implemented. But references non-existent response models and would crash. |
| `GET /api/orders/{id}` — detail with line_items AND status_history | /1.5 | 0.5 | `routers/orders.py:96-106`: Returns order but wraps in `OrderDetailResponse` that doesn't explicitly include line_items/status_history. |
| `GET /api/orders/workflow-stats` — count per status | /1 | 0 | Not implemented. No workflow-stats endpoint exists. |
| `POST /api/orders` — create with line items | /1 | 0.5 | `routers/orders.py:25-77`: Attempts to create order with line items but uses wrong FK names (`purchase_order_id` vs model's `po_id`), missing `datetime` import, doesn't set required financial fields. Would crash. |
| Auto-generate `PO-YYYY-NNN` format | /1 | 0.5 | `routers/orders.py:72`: `po_number = f"PO-{datetime.utcnow().year}-{order.id:03d}"` — uses order.id instead of sequential counter. Close but not spec-compliant. |
| Calculate totals from line items | /1 | 0 | No auto-calculation of subtotal/total from line items in create endpoint. Financial fields are never set. |
| `PUT /api/orders/{id}` — update notes, expected_ready_date | /0.5 | 0 | `routers/orders.py:109-123`: Tries to set `expected_ready_date` which doesn't exist in the model. Would crash. |
| `POST /api/orders/{id}/status` — advance status | /1 | 0.5 | `routers/orders.py:126-176`: Exists but has flawed logic: takes client-provided `current_status_index` instead of determining current status from DB. Does not validate actual current status. |
| Status workflow: all 10 stages defined | /1 | 1 | `routers/orders.py:139-150`: All 10 stages listed: draft, sent, confirmed, in_production, shipped, in_transit, customs, received, qcd, stocked. |
| Forward-only status transitions enforced | /1 | 0.5 | `routers/orders.py:151-153`: Only prevents advancing beyond last status. Does NOT verify the current_status_index matches actual DB status, so backward transitions possible via wrong index. |
| Status change recorded in `po_status_history` | /0.5 | 0.5 | `routers/orders.py:160-169`: Creates POStatusHistory entry, but records `old_status` as the ALREADY UPDATED status (committed before history entry). Bug. |

**Feature Score: 4.0/10**

---

### F08: Payments Router (Weight: 7%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `GET /api/payments` — list with status filter | /1 | 0 | `routers/payments.py:17-31`: Exists but broken imports (`from .. import models, schemas, services, auth`). References `auth.get_current_active_user` which doesn't exist. Non-functional. |
| `GET /api/payments/{id}` — detail | /1 | 0 | `routers/payments.py:34-42`: Exists but broken imports. Non-functional. |
| `GET /api/payments/summary` — monthly totals, YTD, total fees | /2 | 0 | `routers/payments.py:45-62`: Exists but uses `db.func.sum` (wrong API — should be `func.sum` from sqlalchemy). Broken imports. Non-functional. |
| `POST /api/payments` — create with auto-generated PAY-YYYY-NNN | /2 | 0 | `routers/payments.py:65-96`: Exists but references `models.Payment.Status.CREATED` which doesn't exist. Wrong field names. Non-functional. |
| `PUT /api/payments/{id}` — update status/notes, logs to audit_log | /2 | 0 | `routers/payments.py:99-125`: Exists but calls `services.audit.log_action()` which is a pass stub. Broken imports. |
| `GET /api/payments/{id}/sync-wise` — stub returns current data | /1 | 0 | `routers/payments.py:128-137`: Exists but broken imports. Non-functional. |
| Proper Pydantic schemas | /1 | 0.5 | `schemas/payments.py:1-39`: PaymentBase, PaymentCreate, PaymentOut defined. But router references non-existent schemas (PaymentListResponse, PaymentDetailResponse, etc.). |

**Feature Score: 0.5/10**

---

### F09: Inventory Router (Weight: 15%)

#### F09a: Read Endpoints
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `GET /api/inventory/` — list with stock status | /2 | 0 | `routers/inventory.py:17-36`: References `models.Product.stock` (doesn't exist — field is `quantity` on Inventory model, not Product). References non-existent `services.inventory_monitor.get_stock_status`. Broken imports. |
| Status filter query param | /1 | 0 | No `status_filter` parameter on the list endpoint. |
| Uses `joinedload` for product relationship | /0.5 | 0 | No eager loading used. |
| `GET /api/inventory/stats` — aggregate statistics | /1.5 | 0 | `routers/inventory.py:39-49`: References `models.Product.stock` which doesn't exist. Non-functional. |
| `GET /api/inventory/{product_id}` — auto-creates if missing | /1 | 0 | `routers/inventory.py:52-71`: Auto-creates Product (not Inventory). Creates with wrong fields (`stock`, `price`). Non-functional. |
| `GET /api/inventory/{product_id}/movements` — paginated | /1 | 0 | `routers/inventory.py:74-94`: References `models.InventoryMovement` which doesn't exist (should be `StockMovement`). Non-functional. |

#### F09b: Receive Shipment
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `POST /api/inventory/receive-shipment` — accepts bulk items | /1 | 0 | `routers/inventory.py:97-133`: References `models.InventoryMovement` (doesn't exist), `product.stock` (doesn't exist). Non-functional. |
| Validates ALL products exist before processing (atomic) | /1.5 | 0 | No pre-validation. Processes one-by-one and catches exceptions. |
| Creates `IN` stock movement per item | /1 | 0 | Uses `type="RECEIVE"` instead of `StockMovementTypeEnum.IN`. Wrong model reference. |
| Updates inventory quantity and `updated_at` | /0.5 | 0 | References non-existent `product.stock` field. |

#### F09c: Sales
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `POST /api/inventory/sale` — validates sufficient stock | /1 | 0 | Not implemented (file ends after receive-shipment). |
| Creates `OUT` movement + sale record | /1 | 0 | Not implemented. |
| Decrements inventory | /0.5 | 0 | Not implemented. |
| `POST /api/inventory/bulk-sale` — atomic bulk sales | /1.5 | 0 | Not implemented. |
| Insufficient stock returns 400 error | /1 | 0 | Not implemented. |

#### F09d: Adjustment
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `POST /api/inventory/adjust` — absolute target quantity | /1 | 0 | Not implemented. |
| Prevents negative inventory | /1 | 0 | Not implemented. |
| Creates `ADJUSTMENT` movement with correct delta | /1 | 0 | Not implemented. |
| Updates `last_counted_at` | /0.5 | 0 | Not implemented. |

**Subtotal: 0/18 → Normalized: 0.0/10**

**Feature Score: 0.0/10**

---

### F10: Dashboard Router (Weight: 5%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `GET /api/dashboard/` — returns all 4 KPIs | /4 | 0 | No backend dashboard router exists. Only a frontend API route (`frontend/src/app/api/dashboard/route.ts`) that returns hardcoded mock data. |
| `GET /api/dashboard/order-pipeline` — orders grouped by status | /3 | 0 | Not implemented. |
| `GET /api/dashboard/inventory-health` — per-SKU status summary | /3 | 0 | Not implemented. |

**Feature Score: 0.0/10**

---

### F11: Audit Service (Weight: 8%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `log_action` function exists with correct signature | /1 | 0.5 | `services/audit.py:24-39`: Function exists but has different signature than spec (uses `resource_type`/`resource_id` instead of `table_name`/`record_id`). Missing `db`, `request` params. |
| Logs CREATE actions with new_values | /1 | 0 | Function body is `pass` — does nothing (`services/audit.py:39`). |
| Logs UPDATE actions with old AND new values | /1.5 | 0 | Function body is `pass`. |
| Logs DELETE actions | /1 | 0 | Function body is `pass`. |
| Excludes `password_hash` and `mfa_secret` from logged values | /1.5 | 0 | Comment says "Excludes sensitive fields" but implementation is `pass`. |
| Only logs when values actually change (change detection) | /1.5 | 0 | Not implemented. |
| Captures IP address from `request.client.host` | /0.5 | 0 | Not implemented. |
| Captures User-Agent from headers | /0.5 | 0 | Not implemented. |
| Audit logging integrated into all mutation endpoints | /1 | 0 | No routers call the audit service (except payments router which is itself broken). init_db.py creates audit records directly but that's seed data, not runtime integration. |

**Feature Score: 0.5/10**

---

### F12: Inventory Monitor Service (Weight: 6%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `_calculate_days_until_stockout` — queries last 90 days of sales | /2 | 0 | `services/inventory_monitor.py:15-22`: Returns hardcoded `365.0`. No query, no calculation. Comment says "Simplified implementation." |
| `daily_velocity = total_sold / 90` formula correct | /1 | 0 | No formula implemented. |
| Returns infinity when no sales | /1 | 0.5 | Returns 365.0 always, which is a large number but not infinity. Partially handles zero velocity by not dividing. |
| `_should_send_alert` — checks notifications table for 24h dedup | /2 | 0.5 | `services/inventory_monitor.py:24-32`: Checks if 24h passed since `last_alerted_at`, but does NOT query the notifications table. Logic is also incorrect: `datetime.utcnow() > last_24h` is always True. |
| `check_low_stock_and_alert` — iterates LOW/OUT_OF_STOCK products | /2 | 0.5 | `services/inventory_monitor.py:34-72`: Queries products with stock <= reorder_point. But references `Product.stock` which doesn't exist, uses `Product.reorder_point` via getattr with default 10. Logic exists but is non-functional. |
| Creates notification record when alert sent | /1 | 0 | No notification record creation. Code has `pass` placeholder. |
| Integrates with email sending | /1 | 0 | Comment says "TODO: Call email service" (`services/inventory_monitor.py:63`). Not integrated. |

**Feature Score: 1.5/10**

---

### F13: Email Utility (Weight: 3%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `send_low_stock_alert` function with correct params | /2 | 0 | No `utils/email.py` file exists. No email utility at all. |
| HTML email with styled alert card | /3 | 0 | Not implemented. |
| Includes: SKU, current qty, reorder point, days until stockout | /2 | 0 | Not implemented. |
| Action buttons (Create PO, View Inventory) | /1 | 0 | Not implemented. |
| Sends via SMTP | /2 | 0 | Not implemented. |

**Feature Score: 0.0/10**

---

### F14: Background Tasks / Scheduler (Weight: 3%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| Runs `check_low_stock_and_alert()` periodically | /4 | 0 | No `tasks/scheduler.py` file exists. No scheduler. |
| Integrated with FastAPI lifespan | /3 | 0 | Lifespan exists but doesn't start any background tasks (`main.py:22-27`). |
| Error handling (doesn't crash on failure) | /2 | 0 | Not implemented. |
| Graceful shutdown | /1 | 0 | Not implemented. |

**Feature Score: 0.0/10**

---

### F15: Frontend Dashboard (Weight: 7%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| Client component (`'use client'`) | /0.5 | 0.5 | `page.tsx:1`: `'use client';` present. |
| Fetches `GET /api/dashboard/kpis` on mount | /1.5 | 1 | `page.tsx:16-30`: Fetches `/api/dashboard/` (not `/api/dashboard/kpis`). Close but wrong path. Uses Next.js route handler returning mock data. |
| 4 KPI cards displayed | /2 | 2 | `page.tsx:37-53`: All 4 cards: Total Inventory Value (blue), Pending Orders (green), Low Stock Alerts (amber), MTD Revenue (red). |
| Responsive grid (1->2->4 columns) | /1 | 1 | `page.tsx:36`: `grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4`. Correct breakpoints. |
| Loading spinner state | /1 | 1 | `page.tsx:56-60`: Animated spinner shown during loading. |
| Error fallback with zeroes | /1 | 1 | `page.tsx:63-67`: Error state shows fallback message. Initial state uses zeroes. |
| Order Pipeline section | /0.5 | 0.5 | `page.tsx:70-76`: Section exists with placeholder. |
| Inventory Health section | /0.5 | 0 | `page.tsx:79-84`: Section exists but uses HTML comment `<!-- -->` instead of JSX comment `{/* */}` (`page.tsx:82`). This causes a JSX compilation error, making the frontend unable to build. |
| TailwindCSS styling throughout | /1 | 0.5 | Tailwind classes used consistently, but `globals.css:6` has invalid `@tailwind body;` directive which would cause CSS compilation error. |
| Light theme (white bg, dark text) | /0.5 | 0.5 | `page.tsx:33`: `bg-white`, `layout.tsx:16`: `bg-white text-gray-900`. Correct palette: blue/green/amber/red cards. |
| `layout.tsx` and `globals.css` present | /0.5 | 0.5 | Both present: `frontend/src/app/layout.tsx`, `frontend/src/app/globals.css`. |

**Feature Score: 7.5/10**

---

### F16: Infrastructure — Docker & Nginx (Weight: 5%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| 3 services defined (frontend, backend, nginx) | /2 | 2 | `docker-compose.yml:4-43`: All three services defined. |
| Frontend: port 3000, resource limits (0.5 CPU / 512MB) | /1 | 1 | `docker-compose.yml:5-9`: Port 3000, cpus 0.5, mem_limit 512m. |
| Backend: port 8000, resource limits (0.75 CPU / 1GB), health check every 30s | /1.5 | 1.5 | `docker-compose.yml:17-28`: Port 8000, cpus 0.75, mem_limit 1GB, healthcheck interval 30s. |
| Nginx: ports 80 + 443, SSL termination | /1 | 1 | `docker-compose.yml:30-38`: Ports 80 and 443. SSL certs referenced in nginx.conf. |
| Custom bridge network | /0.5 | 0.5 | `docker-compose.yml:42-44`: Custom bridge network `inventory_net`. |
| Backend reads from `.env` file | /0.5 | 0.5 | `docker-compose.yml:23`: `env_file: .env`. |
| Nginx proxies `/api/` to backend:8000 | /1 | 1 | `nginx.conf:21-25`: `location /api/ { proxy_pass http://backend:8000; }`. |
| Nginx proxies `/` to frontend:3000 | /1 | 1 | `nginx.conf:28-32`: `location / { proxy_pass http://frontend:3000; }`. |
| SSL certs in `./certs/` referenced | /0.5 | 0.5 | `docker-compose.yml:37`: `./certs:/etc/nginx/certs`. `nginx.conf:13-14`: SSL cert paths. |
| Compose file is valid and well-structured | /1 | 1 | Valid YAML, properly structured with services, networks, volumes. |

**Feature Score: 10.0/10**

---

### F17: Database Initialization Script (Weight: 4%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| Admin user: admin@example.com / admin123 with bcrypt hash | /2 | 1 | `init_db.py:38-69`: Creates admin@example.com with bcrypt hash. BUT uses `bcrypt.gensalt()` without specifying rounds=12, and script imports `from app.database import Base, engine, SessionLocal` which don't exist. Script would crash. |
| One supplier with contacts | /1.5 | 1.5 | `init_db.py:72-113`: Creates Alpha Toys Ltd supplier with contact Li Wei. Looks correct. |
| 4-8 sample products | /1.5 | 1 | `init_db.py:116-198`: Creates 4 products with proper fields. But `create_sample_products` returns `products_data` (list of dicts) instead of Product objects, breaking downstream usage. |
| 2 purchase orders (one CONFIRMED, one DRAFT) with line items | /2 | 0.5 | `init_db.py:201-257`: Creates only ONE PO (DRAFT status). No CONFIRMED PO. Also uses `products[:2]` on dict list so `product.id` would crash. |
| Inventory records (one healthy, one low stock) | /1.5 | 0.5 | `init_db.py:174-182` and `329-368`: Creates inventory records all with quantity=100 (all healthy). No low stock scenario. Duplicate inventory creation would conflict. |
| Sample stock movements (IN and OUT) | /1.5 | 1 | `init_db.py:341-348`: Creates IN movements. `init_db.py:383-394`: Creates OUT movement via sale. Both types present. |

**Feature Score: 5.5/10**

---

### F18: Testing Suite (Weight: 12%)

#### F18a: Test Infrastructure
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| In-memory SQLite for test isolation | /2 | 0 | No `conftest.py` exists. No test database setup. |
| Autouse fixtures for setup/teardown | /1 | 0 | No conftest.py, no autouse fixtures. |
| Override `get_db()` dependency per test | /1.5 | 0 | No dependency override. |
| pytest-asyncio for async tests | /0.5 | 0 | In pyproject.toml dev deps but no `conftest.py` configuring it. |

#### F18b: Test Coverage by Module
| Test File | Target Count | Max | Score | Evidence |
|-----------|-------------|-----|-------|----------|
| `test_products.py` | 10 | /1 | 0 | File does NOT exist. |
| `test_orders.py` | 9 | /1 | 0 | `tests/test_orders.py`: 9 tests but imports `Order`, `LineItem` from `..models` — these don't exist. No `__init__.py`. All tests would crash at import. |
| `test_payments.py` | 11 | /1 | 0 | `tests/test_payments.py`: 11 tests but imports `Payment`, `Invoice` — non-existent models. All crash at import. |
| `test_inventory.py` | 22 | /1.5 | 0 | `tests/test_inventory.py`: 16 tests but imports `Inventory`, `Product`, `Shipment`, `Sale` with wrong constructors. All crash. Also only 16 tests, not 22. |
| `test_suppliers.py` | 10 | /1 | 0 | `tests/test_suppliers.py`: 9 tests but imports `Supplier`, `Contact` (Contact doesn't exist). All crash. Only 9 tests. |
| `test_auth.py` | 19 | /1.5 | 0 | `tests/test_auth.py`: 15 tests but imports `AuthToken`, `RateLimit` — non-existent. Methods like `.login()`, `.setup_mfa()` don't exist on User model. All crash. Only 15 tests. |
| `test_audit.py` | 18 | /1.5 | 0 | `tests/test_audit.py`: 12 tests. Imports `Order`, `Product`, `AuditLog` — wrong models. References `LineItem` without import. All crash. Only 12 tests. |
| `test_inventory_monitor.py` | 18 | /1.5 | 0 | `tests/test_inventory_monitor.py`: 13 tests. Imports `Inventory`, `Product`, `StockAlert` — non-existent. All crash. Only 13 tests. |

#### F18c: Test Quality
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| Total test count >= 117 | /2 | 0 | 85 tests across 7 files (missing test_products.py). Below target of 117. |
| All tests pass (0 failures) | /3 | 0 | 0% pass rate. ALL tests would crash at import due to: (1) no `__init__.py`, (2) importing non-existent classes, (3) calling non-existent methods. |
| Tests cover edge cases | /2 | 0 | Tests are conceptually written (cover edge cases like rate limiting, deduplication) but none are functional. |
| Tests are independent | /1 | 0 | Cannot evaluate — no tests run. |

**Subtotal: 0/22 → Normalized: 0.0/10**

**Feature Score: 0.0/10**

---

### F19: Security Implementation (Weight: 8%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| Passwords: bcrypt with work factor 12 | /1.5 | 0.5 | `routers/auth.py:16`: `CryptContext(schemes=["bcrypt"], deprecated=False)` — bcrypt used but no `rounds=12` specified. Uses passlib default (12 in some versions, but not explicitly set). |
| JWT: HS256 algorithm | /0.5 | 0 | No JWT implementation. Config has `ALGORITHM` var but no encoding/decoding. |
| JWT: 15-min access tokens | /0.5 | 0 | No JWT implementation. |
| JWT: 7-day refresh tokens | /0.5 | 0 | No JWT implementation. |
| MFA: TOTP via pyotp | /1 | 1 | `routers/auth.py:7,110-118`: pyotp imported and used for TOTP setup and verification. |
| Rate limiting: 100/min global | /0.5 | 0 | Middleware is not a proper class. Would crash on startup. |
| Rate limiting: 5/min on auth | /0.5 | 0 | Middleware is not a proper class. Would crash on startup. |
| Input sanitization: bleach for HTML | /1 | 0 | No bleach usage anywhere. Not in dependencies. |
| Encrypted storage: Fernet for sensitive fields | /1.5 | 0 | No Fernet usage anywhere. Not in dependencies. Config has `DB_ENCRYPTION_KEY` but no implementation. |
| No raw SQL with user input (ORM only) | /1 | 1 | ORM used throughout routers. No `text()` with user input in routers. |
| CORS: whitelist only (not wildcard) | /0.5 | 0.5 | `main.py:37`: Specific origins whitelisted, not `*`. |
| Security headers (all 4) | /1 | 0 | Middleware would crash. Even if functional: 3 of 4 headers (missing CSP, has X-XSS-Protection instead). |

**Feature Score: 3.0/10**

---

### F20: Business Logic Correctness (Weight: 7%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| Inventory Status Rules correct | /2 | 0 | No stock status calculation in any functional endpoint. Inventory router references non-existent `services.inventory_monitor.get_stock_status`. |
| Days Until Stockout formula | /1.5 | 0 | `services/inventory_monitor.py:15-22`: Returns hardcoded 365.0. No formula. |
| Alert Deduplication | /1.5 | 0 | `services/inventory_monitor.py:24-32`: Logic exists but is incorrect (`datetime.utcnow() > last_24h` always True) and doesn't query notifications table. |
| PO Number Generation — PO-YYYY-NNN | /1 | 0.5 | `routers/orders.py:72`: Uses `order.id` not sequential counter. Format is close but not auto-incrementing within year. |
| Payment ID Generation — PAY-YYYY-NNN | /1 | 0 | `routers/payments.py:82`: `f"PAY-{datetime.utcnow().year}-{payment.id}"` — uses payment.id not NNN format. Router is also broken. |
| Atomic Bulk Operations | /2 | 0 | No functional receive-shipment or bulk-sale. Receive-shipment doesn't pre-validate. No bulk-sale endpoint. |
| Database transactions — proper rollback on failure | /1 | 0.5 | `routers/inventory.py:105-129`: Has try/rollback pattern for receive-shipment. `init_db.py:481-482`: Has rollback. But neither works due to other bugs. |

**Feature Score: 1.0/10**

---

## Final Score Card

| Feature | Weight | Score |
|---------|--------|-------|
| F01: Project Structure & Setup | 5% | 6.5 |
| F02: Database Schema & Models | 10% | 8.0 |
| F03: App Setup & Configuration | 5% | 4.5 |
| F04: Auth Router | 8% | 4.0 |
| F05: Products Router | 7% | 6.0 |
| F06: Suppliers Router | 5% | 1.0 |
| F07: Purchase Orders Router | 10% | 4.0 |
| F08: Payments Router | 7% | 0.5 |
| F09: Inventory Router | 15% | 0.0 |
| F10: Dashboard Router | 5% | 0.0 |
| F11: Audit Service | 8% | 0.5 |
| F12: Inventory Monitor Service | 6% | 1.5 |
| F13: Email Utility | 3% | 0.0 |
| F14: Background Tasks | 3% | 0.0 |
| F15: Frontend Dashboard | 7% | 7.5 |
| F16: Infrastructure (Docker/Nginx) | 5% | 10.0 |
| F17: DB Init Script | 4% | 5.5 |
| F18: Testing Suite | 12% | 0.0 |
| F19: Security Implementation | 8% | 3.0 |
| F20: Business Logic Correctness | 7% | 1.0 |
| **Weighted Total** | **100%** | **4.26** |

### Weighted Calculation
(6.5*0.05) + (8.0*0.10) + (4.5*0.05) + (4.0*0.08) + (6.0*0.07) + (1.0*0.05) + (4.0*0.10) + (0.5*0.07) + (0.0*0.15) + (0.0*0.05) + (0.5*0.08) + (1.5*0.06) + (0.0*0.03) + (0.0*0.03) + (7.5*0.07) + (10.0*0.05) + (5.5*0.04) + (0.0*0.12) + (3.0*0.08) + (1.0*0.07)
= 0.325 + 0.800 + 0.225 + 0.320 + 0.420 + 0.050 + 0.400 + 0.035 + 0.000 + 0.000 + 0.040 + 0.090 + 0.000 + 0.000 + 0.525 + 0.500 + 0.220 + 0.000 + 0.240 + 0.070
= **4.26/10**

---

## Critical Defects Summary

1. **No `__init__.py` files** — Python package imports completely broken
2. **`database.py` missing `Base`, `get_db`, `SessionLocal`** — app cannot start
3. **Models have indexes on wrong tables** — would crash at table creation
4. **Product, PurchaseOrder, Inventory missing primary key `id` columns**
5. **Main.py imports from wrong module paths** (`.auth` vs `.routers.auth`)
6. **Middleware not implemented as proper classes** — `app.add_middleware()` crashes
7. **All 85 tests import non-existent classes** — 0% pass rate
8. **No JWT token generation** — auth endpoint returns plain JSON
9. **Suppliers, Payments, Inventory routers have broken imports/references**
10. **Frontend JSX has HTML comment (`<!-- -->`)** — won't compile
