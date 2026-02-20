# Relative Grading Report: OpenCode Kimi

## Summary
- **Weighted Average: 8.76/10**
- **Grade: A**

---

## Feature Scores

### F01: Project Structure & Setup (Weight: 5%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| Directory structure matches spec | /2 | 2 | `backend/app/routers/`, `backend/app/services/`, `backend/app/utils/`, `backend/app/tasks/`, `frontend/src/app/`, `infrastructure/` all present |
| `pyproject.toml` with all dependencies | /2 | 2 | `pyproject.toml:6-23`: fastapi, sqlalchemy, passlib[bcrypt], python-jose, pyotp, slowapi, bleach, cryptography, pydantic-settings, uvicorn, alembic all listed |
| `alembic.ini` + alembic directory exists | /1 | 1 | `alembic.ini` present and configured |
| `uv` used for package management | /1 | 0 | No `uv.lock` found anywhere in project. No evidence of uv usage |
| Frontend `package.json` with correct deps | /1 | 1 | `package.json`: Next.js 14.2.35, React ^18, TailwindCSS ^3.4.1 |
| `tsconfig.json` and `tailwind.config.ts` | /1 | 1 | Both present and properly configured |
| `.env` example or config documentation | /1 | 1 | `infrastructure/.env.example` with all config vars documented |
| Clean separation of concerns | /1 | 1 | routers/, schemas/, services/, utils/, tasks/ all properly separated |

**Feature Score: 9/10**

---

### F02: Database Schema & Models (Weight: 10%)

#### F02a: Auth & Security Tables
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `users` table — all fields | /2 | 2 | `models.py:86-97`: email, password_hash, mfa_secret, mfa_enabled, role (UserRole enum), is_active |
| `audit_log` table — all fields including JSON old/new values | /2 | 2 | `models.py:106-121`: table_name, record_id, action, old_values (JSON), new_values (JSON), user_id FK, ip_address, user_agent, created_at |

#### F02b: Products & Suppliers Tables
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `suppliers` — all fields | /1 | 1 | `models.py:125-138`: name, legal_name, website, address, payment_terms, lead_time_days, moq, notes |
| `supplier_contacts` — FK to supplier, platform enum | /1 | 0.5 | `models.py:150-162`: supplier_id FK present, but `platform` is `String(50)` not a platform enum |
| `products` — all fields with constraints | /2 | 2 | `models.py:165-191`: internal_sku (unique), supplier_sku, age_range_start/end, price_usd, units_per_carton, is_best_seller, is_active, supplier_id FK |
| `price_history` — all fields | /1 | 1 | `models.py:194-205`: product_id FK, supplier_id FK, price_usd, effective_date |

#### F02c: Purchase Orders Tables
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `purchase_orders` — all fields | /2 | 2 | `models.py:209-235`: po_number (unique), status (POStatus enum), subtotal_usd, shipping_usd, other_fees_usd, total_usd |
| `po_line_items` — FKs and fields | /1 | 1 | `models.py:238-249`: po_id FK, product_id FK, quantity, unit_price_usd |
| `po_status_history` — all fields | /1 | 1 | `models.py:252-265`: po_id FK, old_status, new_status, changed_by FK, changed_at |

#### F02d: Shipments Tables
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `shipments` — all fields | /1 | 1 | `models.py:269-287`: po_id FK, carrier, tracking_number, bill_of_lading, origin_port, destination_port, status |
| `shipment_milestones` — FK, enum, dates | /1 | 1 | `models.py:290-301`: shipment_id FK, milestone_type (MilestoneType enum), expected_date, actual_date |

#### F02e: Payments Table
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `payments` — all fields | /2 | 2 | `models.py:305-326`: payment_id (unique), po_id FK, supplier_id FK, wise_transfer_id, wise_tracking_id, status (PaymentStatus enum), amount_usd, fee_usd, amount_received, currency_received |

#### F02f: Inventory Tables
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `inventory` — all fields with defaults | /2 | 2 | `models.py:330-342`: product_id (unique FK), quantity (default=0), reorder_point (default=10), safety_stock_days (default=14), last_counted_at |
| `stock_movements` — enums and references | /1 | 1 | `models.py:345-358`: movement_type (MovementType enum), reference_type (ReferenceType enum), reference_id |
| `sales` — platform enum, order_reference | /1 | 1 | `models.py:361-374`: product_id FK, platform (Platform enum), order_reference |

#### F02g: Planning Tables
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `seasonality_factors` — month, multiplier | /1 | 1 | `models.py:378-385`: month (unique), multiplier |
| `holidays` — country, name, dates, type enum | /1 | 1 | `models.py:388-397`: country, holiday_name, start/end_date, holiday_type (HolidayType enum) |
| `settings` — key/value store | /0.5 | 0.5 | `models.py:400-407`: key (unique), value |
| `notifications` — product_id FK, alert_type, message, status enum | /0.5 | 0.5 | `models.py:410-424`: product_id FK, alert_type, message, status (NotificationStatus enum) |

#### F02h: Indexes & Constraints
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| All specified indexes created (11 total) | /3 | 2 | 10 explicit Index() calls at `models.py:428-445`. Missing 1 of 11 specified indexes |
| Unique constraints on internal_sku, po_number, payment_id | /2 | 2 | `models.py:169`: internal_sku unique; `models.py:213`: po_number unique; `models.py:309`: payment_id unique |
| Foreign key relationships correctly defined | /2 | 2 | All FKs present and correct across all models |
| Proper use of enums for status fields | /1 | 1 | 10 enums: UserRole, POStatus, PaymentStatus, MovementType, ReferenceType, Platform, NotificationStatus, HolidayType, MilestoneType + more |
| Table count >= 20 | /2 | 1 | 19 tables. 15-19 range → 1 point |

**Subtotal: 31.5/34 → Normalized: 9.3/10**

**Feature Score: 9.3/10**

---

### F03: Application Setup & Configuration (Weight: 5%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| FastAPI app with lifespan manager | /1 | 0.5 | `main.py:38-44`: lifespan exists but only prints start/shutdown. Scheduler NOT integrated — no background tasks started |
| CORS configured for specified origins | /1 | 1 | `main.py:63-79`: specific origins whitelisted + Tailscale regex. Not wildcard |
| Global rate limiter: 100 req/min | /1 | 1 | `main.py:35`: `Limiter(key_func=get_remote_address, default_limits=["100/minute"])` |
| Auth endpoint rate limit: 5 req/min | /1 | 0 | NOT implemented. No `@limiter.limit` decorator on auth endpoints |
| Security headers middleware (all 4 headers) | /2 | 1.5 | `main.py:25-31`: X-Content-Type-Options, X-Frame-Options, CSP present. **Missing Referrer-Policy** (only in nginx.conf:19). 3/4 required headers |
| Health check `GET /health` returns `{"status": "ok"}` | /1 | 1 | `main.py:82-84`: returns `{"status": "ok"}` |
| Pydantic Settings with all env vars | /2 | 2 | `config.py:5-33`: DATABASE_URL, SECRET_KEY, DB_ENCRYPTION_KEY, ALGORITHM, token expiry, WISE, SMTP, DEBUG, Tailscale |
| `app/database.py` — proper session management, `get_db` dependency | /1 | 1 | Correct pattern: engine, SessionLocal, Base, get_db() generator |

**Feature Score: 8/10**

---

### F04: Auth Router (Weight: 8%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `POST /api/auth/register` — creates user, hashes password with bcrypt | /2 | 2 | `auth.py:134-150`: register endpoint; `auth.py:66`: `pwd_context.hash(password, rounds=12)` — bcrypt work factor 12 |
| `POST /api/auth/login` — verifies credentials, returns JWT access + refresh tokens | /2 | 2 | `auth.py:153-178`: returns access_token, refresh_token, token_type |
| `POST /api/auth/refresh` — exchanges refresh token for new access token | /1 | 1 | `auth.py:181-222`: validates refresh, invalidates old, issues new pair |
| `POST /api/auth/mfa/setup` — generates TOTP secret, returns QR URI | /1.5 | 1.5 | `auth.py:225-266`: pyotp.random_base32(), provisioning_uri, QR as base64 SVG |
| `POST /api/auth/mfa/verify` — validates TOTP code, enables MFA | /1.5 | 1.5 | `auth.py:269-304`: validates TOTP, sets mfa_enabled=True, returns tokens |
| JWT uses HS256 with 15-min access token expiry | /1 | 1 | `config.py:9`: ALGORITHM="HS256"; `config.py:11`: ACCESS_TOKEN_EXPIRE_MINUTES=15 |
| Refresh token: 7-day expiry | /0.5 | 0.5 | `config.py:12`: REFRESH_TOKEN_EXPIRE_DAYS=7 |
| Rate limiting: 5 req/min on auth endpoints | /0.5 | 0 | NOT implemented |

**Feature Score: 9.5/10**

---

### F05: Products Router (Weight: 7%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `GET /api/products` — list with filtering (best_seller, active) | /2 | 2 | `products.py:14-37`: both `best_seller` and `is_active` Query params |
| `GET /api/products/{id}` — detail with supplier info | /1 | 1 | `products.py:40-53`: supplier via relationship (`models.py:183`) |
| `POST /api/products` — create product, record initial price in price_history | /2 | 2 | `products.py:56-111`: creates product + PriceHistory if supplier_id present |
| `PUT /api/products/{id}` — update; records price_history if price changed | /2 | 2 | `products.py:114-176`: detects price change at line 147-149, records PriceHistory |
| `DELETE /api/products/{id}` — soft delete (is_active=False) | /1.5 | 1.5 | `products.py:179-197`: sets is_active=False |
| Proper Pydantic schemas for request/response | /1 | 1 | `schemas/products.py`: ProductCreate, ProductUpdate, ProductResponse, ProductDetailResponse |
| Input validation (e.g., valid SKU format) | /0.5 | 0.5 | `products.py:66-72`: duplicate SKU check; `products.py:129-144`: uniqueness on update |

**Feature Score: 10/10**

---

### F06: Suppliers Router (Weight: 5%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `GET /api/suppliers` — list all | /1.5 | 1.5 | GET list endpoint with SupplierListResponse (items + total) |
| `GET /api/suppliers/{id}` — detail with contacts | /2 | 2 | Detail includes contacts via relationship |
| `GET /api/suppliers/{id}/contacts` — contacts only | /1.5 | 1.5 | Dedicated contacts endpoint present |
| `POST /api/suppliers` — create | /2 | 2 | Create with validation |
| `PUT /api/suppliers/{id}` — update | /2 | 2 | Update with validation |
| Proper Pydantic schemas | /1 | 1 | Well-defined schemas for all operations |

**Feature Score: 10/10**

---

### F07: Purchase Orders Router (Weight: 10%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `GET /api/orders` — list with filters (status, supplier_id) | /1 | 1 | `orders.py:74-99`: both status and supplier_id Query params |
| `GET /api/orders/{id}` — detail with line_items AND status_history | /1.5 | 1.5 | `orders.py:102-116`: order loaded with line_items and status_history relationships |
| `GET /api/orders/workflow-stats` — count per status | /1 | 1 | `orders.py:119-164`: groups by all 10 statuses with counts |
| `POST /api/orders` — create with line items | /1 | 1 | `orders.py:167-249`: creates PO + line items in transaction |
| Auto-generate `PO-YYYY-NNN` format | /1 | 1 | `orders.py:48-71`: generate_po_number() with auto-increment |
| Calculate totals from line items | /1 | 1 | `orders.py:187-189`: `subtotal = sum(qty * unit_price_usd)` |
| `PUT /api/orders/{id}` — update notes | /0.5 | 0.5 | `orders.py:252-288`: updates notes |
| `POST /api/orders/{id}/status` — advance status | /1 | 1 | `orders.py:291-349`: validates and advances |
| Status workflow: all 10 stages defined | /1 | 1 | `orders.py:34-45`: DRAFT→SENT→CONFIRMED→IN_PRODUCTION→SHIPPED→IN_TRANSIT→CUSTOMS→RECEIVED→QCD→STOCKED |
| Forward-only status transitions enforced | /1 | 1 | `orders.py:310-321`: checks VALID_TRANSITIONS, returns 400 on invalid |
| Status change recorded in `po_status_history` | /0.5 | 0.5 | `orders.py:327-332`: creates POStatusHistory with old/new status |

**Feature Score: 10/10**

---

### F08: Payments Router (Weight: 7%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `GET /api/payments` — list with status filter | /1 | 1 | List endpoint with status, supplier_id, po_id filters |
| `GET /api/payments/{id}` — detail | /1 | 1 | Detail endpoint present |
| `GET /api/payments/summary` — monthly totals, YTD, total fees | /2 | 2 | Summary with monthly breakdown, YTD total, total fees |
| `POST /api/payments` — create with auto-generated PAY-YYYY-NNN | /2 | 2 | Auto PAY-YYYY-NNN with auto-increment |
| `PUT /api/payments/{id}` — update status/notes, logs to audit_log | /2 | 2 | Updates + calls log_action for audit |
| `GET /api/payments/{id}/sync-wise` — stub | /1 | 1 | Stub endpoint exists |
| Proper Pydantic schemas | /1 | 1 | Well-defined schemas in `schemas/payments.py` |

**Feature Score: 10/10**

---

### F09: Inventory Router (Weight: 15%)

#### F09a: Read Endpoints
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `GET /api/inventory/` — list all with stock status | /2 | 2 | `inventory.py:109-148`: computes stock_status via `get_stock_status()` |
| Status filter query param (`?status_filter=LOW`) | /1 | 0.5 | `inventory.py:111`: uses `status` query param instead of `status_filter`. Functionally works but wrong param name |
| Uses `joinedload` for product relationship | /0.5 | 0 | `inventory.py:120`: uses `db.query(Inventory).join(Product)` — SQL JOIN, not SQLAlchemy `joinedload` for eager loading |
| `GET /api/inventory/stats` — aggregate statistics | /1.5 | 1.5 | `inventory.py:151-184`: total_products, total_value, low_stock_count, out_of_stock_count, ok_stock_count |
| `GET /api/inventory/{product_id}` — detail; auto-creates if missing | /1 | 1 | `inventory.py:187-238`: auto-creates with qty=0, reorder_point=10, safety_stock_days=14 |
| `GET /api/inventory/{product_id}/movements` — paginated history | /1 | 1 | `inventory.py:241-256`: paginated via `limit` query param |

#### F09b: Receive Shipment
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `POST /api/inventory/receive-shipment` — accepts bulk items + reference | /1 | 1 | `inventory.py:259-345`: ReceiveShipmentRequest with po_id and items list |
| Validates ALL products exist before processing (atomic) | /1.5 | 0.5 | `inventory.py:269-279`: processes one-by-one with `continue` on not found. NOT pre-validated atomically |
| Creates `IN` stock movement per item | /1 | 1 | `inventory.py:304-312`: StockMovement with MovementType.IN |
| Updates inventory quantity and `updated_at` | /0.5 | 0.5 | `inventory.py:299-301`: quantity += item.quantity, last_counted_at updated |

#### F09c: Sales
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `POST /api/inventory/sale` — validates sufficient stock | /1 | 1 | `inventory.py:373-377`: checks qty < sale_data.quantity, returns 400 |
| Creates `OUT` movement + sale record | /1 | 1 | `inventory.py:385-402`: Sale record + StockMovement(OUT) |
| Decrements inventory | /0.5 | 0.5 | `inventory.py:381`: `inventory.quantity -= sale_data.quantity` |
| `POST /api/inventory/bulk-sale` — atomic bulk sales | /1.5 | 1.5 | `inventory.py:438-489`: first pass validates all, returns 400 if any fail; second pass processes |
| Insufficient stock returns 400 error | /1 | 1 | `inventory.py:374-377`: HTTP_400_BAD_REQUEST |

#### F09d: Adjustment
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `POST /api/inventory/adjust` — accepts absolute target quantity | /1 | 1 | `inventory.py:558-626`: `inventory.quantity = request.new_quantity` |
| Prevents negative inventory | /1 | 1 | `schemas/inventory.py:197`: `new_quantity: int = Field(..., ge=0)` |
| Creates `ADJUSTMENT` movement with correct delta | /1 | 1 | `inventory.py:594-598`: `quantity=abs(request.new_quantity - old_quantity)`, MovementType.ADJUSTMENT |
| Updates `last_counted_at` | /0.5 | 0.5 | `inventory.py:591`: `inventory.last_counted_at = datetime.utcnow()` |

**Subtotal: 17.5/19.5 → Normalized: 9.0/10**

**Feature Score: 9.0/10**

---

### F10: Dashboard Router (Weight: 5%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `GET /api/dashboard/` — returns all 4 KPIs | /4 | 4 | Returns total_inventory_value, pending_orders, low_stock_alerts, mtd_revenue |
| `GET /api/dashboard/order-pipeline` — orders grouped by status | /3 | 3 | Groups by all 10 PO statuses with counts |
| `GET /api/dashboard/inventory-health` — per-SKU status summary | /3 | 3 | Per-SKU data with velocity and days_until_stockout |

**Feature Score: 10/10**

---

### F11: Audit Service (Weight: 8%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `log_action` function exists with correct signature | /1 | 1 | `audit.py:13-22`: all params (db, user_id, table_name, record_id, action, old_values, new_values, request) |
| Logs CREATE actions with new_values | /1 | 1 | `audit.py:56-70`: creates AuditLog with new_filtered |
| Logs UPDATE actions with old AND new values | /1.5 | 1.5 | `audit.py:40-41`: filters both old_values and new_values, both stored |
| Logs DELETE actions | /1 | 1 | Service accepts "DELETE" action. Function correctly processes it |
| Excludes `password_hash` and `mfa_secret` from logged values | /1.5 | 1.5 | `audit.py:10`: `EXCLUDED_FIELDS = {"password_hash", "mfa_secret", "password", "secret"}` |
| Only logs when values actually change (change detection) | /1.5 | 1.5 | `audit.py:44-45`: `if action == "UPDATE" and old_filtered == new_filtered: return None` |
| Captures IP address from `request.client.host` | /0.5 | 0.25 | `audit.py:50-51`: code exists but most callers pass `request=None` (e.g., `orders.py:239`, `inventory.py:315`) |
| Captures User-Agent from headers | /0.5 | 0.25 | `audit.py:52-53`: code exists, same issue — request rarely passed |
| Audit logging integrated into all mutation endpoints | /1 | 0.5 | Orders: CREATE/UPDATE/STATUS logged. Inventory: all mutations logged. **Products and Suppliers do NOT call log_action** |

**Feature Score: 8.5/10**

---

### F12: Inventory Monitor Service (Weight: 6%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `_calculate_days_until_stockout` — queries last 90 days of sales | /2 | 2 | Queries Sale table for last 90 days, sums quantities |
| `daily_velocity = total_sold / 90` formula correct | /1 | 1 | `daily_velocity = total_sold / 90.0` |
| Returns infinity when no sales | /1 | 1 | Returns `float("inf")` for zero velocity |
| `_should_send_alert` — checks notifications table for 24h dedup | /2 | 2 | Queries notifications for same product within 24h |
| `check_low_stock_and_alert` — iterates LOW/OUT_OF_STOCK products | /2 | 2 | Filters by LOW and OUT_OF_STOCK status |
| Creates notification record when alert sent | /1 | 1 | Creates Notification with PENDING, updates to SENT after email |
| Integrates with email sending | /1 | 1 | Calls `send_low_stock_alert()` from utils/email.py |

**Feature Score: 10/10**

---

### F13: Email Utility (Weight: 3%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| `send_low_stock_alert` function with correct params | /2 | 2 | All params: product_sku, current_qty, reorder_point, days_until_stockout, recipients |
| HTML email with styled alert card | /3 | 3 | Well-formatted HTML with styled card, colors, layout |
| Includes: SKU, current qty, reorder point, days until stockout | /2 | 2 | All 4 fields in email body |
| Action buttons (Create PO, View Inventory) | /1 | 1 | Buttons present with links |
| Sends via SMTP | /2 | 2 | Uses SMTP config from settings |

**Feature Score: 10/10**

---

### F14: Background Tasks / Scheduler (Weight: 3%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| Runs `check_low_stock_and_alert()` periodically | /4 | 2 | `scheduler.py`: TaskScheduler runs every 3600s (1 hour). Correct interval but NOT actually running |
| Integrated with FastAPI lifespan | /3 | 0 | `main.py:38-44`: lifespan only prints messages. Scheduler NOT imported or started. Zero integration |
| Error handling (doesn't crash on failure) | /2 | 2 | `scheduler.py`: try/except with logging |
| Graceful shutdown | /1 | 1 | `scheduler.py`: stop() method sets running=False |

**Feature Score: 5/10**

---

### F15: Frontend Dashboard (Weight: 7%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| Client component (`'use client'`) | /0.5 | 0.5 | `page.tsx`: `'use client'` at top |
| Fetches `GET /api/dashboard/` on mount | /1.5 | 1.5 | `page.tsx`: useEffect calls `fetchDashboard()`; `api.ts:19`: fetches `/api/dashboard/` |
| 4 KPI cards displayed | /2 | 2 | Inventory Value (blue), Pending Orders (amber), Low Stock (red), MTD Revenue (green) |
| Responsive grid (1->2->4 columns) | /1 | 1 | `grid-cols-1 sm:grid-cols-2 lg:grid-cols-4` |
| Loading spinner state | /1 | 1 | Loading state with spinner while fetching |
| Error fallback with zeroes | /1 | 1 | Catches errors, shows zeroes |
| Order Pipeline section | /0.5 | 0.5 | Section exists ("Coming soon" placeholder) |
| Inventory Health section | /0.5 | 0.5 | Section exists (placeholder) |
| TailwindCSS styling throughout | /1 | 1 | Consistent Tailwind utility classes |
| Light theme (white bg, dark text) | /0.5 | 0.5 | `globals.css`: white (#ffffff), dark (#333333) |
| `layout.tsx` and `globals.css` present | /0.5 | 0.5 | Both present and configured |

**Feature Score: 10/10**

---

### F16: Infrastructure — Docker & Nginx (Weight: 5%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| 3 services defined (frontend, backend, nginx) | /2 | 1 | `docker-compose.yml`: 4 services (frontend, backend, db, nginx). Has extra `db` service beyond spec's 3 |
| Frontend: port 3000, resource limits | /1 | 1 | `docker-compose.yml:10`: 18374:3000; `docker-compose.yml:16-19`: cpus 0.5, memory 512M |
| Backend: port 8000, resource limits, health check | /1.5 | 1.5 | `docker-compose.yml:35`: 29384:8000; cpus 0.75, memory 1G; health check every 30s |
| Nginx: ports 80 + 443, SSL termination | /1 | 0.5 | `docker-compose.yml:83-85`: 8473:80, 8474:443. Certs mounted but NO HTTPS server block in nginx.conf |
| Custom bridge network | /0.5 | 0.5 | `docker-compose.yml:96-98`: `inventory-network` bridge driver |
| Backend reads from `.env` file | /0.5 | 0.5 | `docker-compose.yml:37-38`: `env_file: - .env` |
| Nginx proxies `/api/` to backend:8000 | /1 | 1 | `nginx.conf:53-65`: `location /api/ { proxy_pass http://backend/; }` |
| Nginx proxies `/` to frontend:3000 | /1 | 1 | `nginx.conf:68-78`: `location / { proxy_pass http://frontend/; }` |
| SSL certs in `./certs/` referenced | /0.5 | 0.5 | `docker-compose.yml:88`: `./certs:/etc/nginx/certs:ro` |
| Compose file is valid and well-structured | /1 | 1 | Valid YAML, proper depends_on, networks, volumes |

**Feature Score: 8.5/10**

---

### F17: Database Initialization Script (Weight: 4%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| Admin user: admin@example.com / admin123 with bcrypt hash | /2 | 2 | `init_db.py`: admin@example.com with bcrypt-hashed "admin123" + 3 more users |
| One supplier with contacts | /1.5 | 1.5 | 3 suppliers with contacts (exceeds requirement) |
| 4-8 sample products | /1.5 | 1.5 | 36 products (12 templates × 3 suppliers) with LUMI-XXXX SKUs |
| 2 purchase orders with line items | /2 | 2 | 16 POs across all 10 statuses with line items and status history |
| Inventory records (one healthy, one low stock) | /1.5 | 1.5 | Varied levels: healthy (50+), low (3-8), zero |
| Sample stock movements (IN and OUT) | /1.5 | 1.5 | IN, OUT, and ADJUSTMENT movements all present |

**Feature Score: 10/10**

---

### F18: Testing Suite (Weight: 12%)

#### F18a: Test Infrastructure
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| In-memory SQLite for test isolation | /2 | 2 | `conftest.py:24`: `TEST_DATABASE_URL = "sqlite:///:memory:"` |
| Autouse fixtures for setup/teardown | /1 | 1 | `conftest.py:44-58`: db_session creates/drops all tables per function |
| Override `get_db()` dependency per test | /1.5 | 1.5 | `conftest.py:61-77`: proper `app.dependency_overrides[get_db]` |
| pytest-asyncio for async tests | /0.5 | 0.5 | `pyproject.toml:29,35`: pytest-asyncio + `asyncio_mode = "auto"` |

#### F18b: Test Coverage by Module
| Test File | Target | Max | Score | Evidence |
|-----------|--------|-----|-------|----------|
| `test_products.py` | 10 | /1 | 0.5 | 9 tests, some use wrong field names — estimated 5-7 passing |
| `test_orders.py` | 9 | /1 | 0 | 11 tests but most are skeletal `pass` stubs; only 1-2 make real API calls |
| `test_payments.py` | 11 | /1 | 0 | 13 tests but nearly all skeletal `pass` stubs with no real API calls |
| `test_inventory.py` | 22 | /1.5 | 0 | 22 tests but nearly all skeletal `pass` stubs with trivial local assertions |
| `test_suppliers.py` | 10 | /1 | 0.5 | 11 tests, some use wrong field names — estimated 5-7 passing |
| `test_auth.py` | 19 | /1.5 | 0.5 | 10 tests (below 19 target); rate limit test expects 429 without auth limiting |
| `test_audit.py` | 18 | /1.5 | 0.5 | 10 real tests using db_session + log_action directly |
| `test_inventory_monitor.py` | 18 | /1.5 | 1 | 12 real tests with db_session — substantive |

#### F18c: Test Quality
| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| Total test count >= 117 | /2 | 0.5 | ~98 tests defined. 40-79 real/substantive tests |
| All tests pass (0 failures) | /3 | 1 | Estimated ~85-90% pass rate: stubs pass, but wrong field names in products/suppliers cause failures. 80%+ → 1 |
| Tests cover edge cases | /2 | 1 | Audit+monitor tests have edge cases (no-op, zero velocity, 24h dedup). Products/suppliers/orders tests mostly stub or happy path |
| Tests are independent (no order dependency) | /1 | 1 | Each test gets fresh DB via create_all/drop_all per function |

**Subtotal: 11.5/22 → Normalized: 5.2/10**

**Feature Score: 5.2/10**

---

### F19: Security Implementation (Weight: 8%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| Passwords: bcrypt with work factor 12 | /1.5 | 1.5 | `auth.py:21`: CryptContext(["bcrypt"]); `auth.py:66`: rounds=12 |
| JWT: HS256 algorithm | /0.5 | 0.5 | `config.py:9`: ALGORITHM="HS256" |
| JWT: 15-min access tokens | /0.5 | 0.5 | `config.py:11`: ACCESS_TOKEN_EXPIRE_MINUTES=15 |
| JWT: 7-day refresh tokens | /0.5 | 0.5 | `config.py:12`: REFRESH_TOKEN_EXPIRE_DAYS=7 |
| MFA: TOTP via pyotp | /1 | 1 | `auth.py:8`: import pyotp; setup + verify endpoints |
| Rate limiting: 100/min global | /0.5 | 0.5 | `main.py:35`: slowapi 100/minute |
| Rate limiting: 5/min on auth | /0.5 | 0 | NOT implemented |
| Input sanitization: bleach for HTML | /1 | 0 | Listed in `pyproject.toml:20` but NEVER imported or used in code |
| Encrypted storage: Fernet for sensitive fields | /1.5 | 0 | cryptography in deps but Fernet NEVER used. Sensitive fields stored plaintext |
| No raw SQL with user input (ORM only) | /1 | 1 | ORM throughout. No `text()` calls in routers |
| CORS: whitelist only (not wildcard) | /0.5 | 0.5 | `main.py:63-74`: specific origins, not `*` |
| Security headers (all 4) | /1 | 0.75 | 3/4: X-Content-Type-Options ✓, X-Frame-Options ✓, CSP ✓. Referrer-Policy MISSING from backend |

**Feature Score: 6.75/10**

---

### F20: Business Logic Correctness (Weight: 7%)

| Criterion | Max | Score | Evidence |
|-----------|-----|-------|----------|
| Inventory Status Rules correct | /2 | 1 | `inventory.py:99-106`: OUT_OF_STOCK uses `qty <= 0` (spec: `qty == 0`), LOW and OK correct. Two of three match spec |
| Days Until Stockout formula | /1.5 | 1.5 | `inventory.py:59-70`: 90-day window, `daily_velocity = total_sold / 90.0`, `days = qty / velocity` |
| Alert Deduplication | /1.5 | 1.5 | inventory_monitor: checks notifications for same product within 24h |
| PO Number Generation | /1 | 1 | `orders.py:48-71`: PO-YYYY-NNN with auto-increment |
| Payment ID Generation | /1 | 1 | Payments router: PAY-YYYY-NNN with auto-increment |
| Atomic Bulk Operations | /2 | 1 | bulk-sale: pre-validates all, 400 if any fail — ATOMIC ✓. receive-shipment: `continue` on missing product — NOT atomic ✗ |
| Database transactions — proper rollback on failure | /1 | 1 | `inventory.py:340-344`: rollback in receive-shipment; `inventory.py:550-554`: rollback in bulk-sale |

**Feature Score: 8.0/10**

---

## Summary Score Sheet

| Feature | Weight | Score | Weighted |
|---------|--------|-------|----------|
| F01: Project Structure & Setup | 5% | 9.0 | 0.450 |
| F02: Database Schema & Models | 10% | 9.3 | 0.930 |
| F03: App Setup & Configuration | 5% | 8.0 | 0.400 |
| F04: Auth Router | 8% | 9.5 | 0.760 |
| F05: Products Router | 7% | 10.0 | 0.700 |
| F06: Suppliers Router | 5% | 10.0 | 0.500 |
| F07: Purchase Orders Router | 10% | 10.0 | 1.000 |
| F08: Payments Router | 7% | 10.0 | 0.700 |
| F09: Inventory Router | 15% | 9.0 | 1.350 |
| F10: Dashboard Router | 5% | 10.0 | 0.500 |
| F11: Audit Service | 8% | 8.5 | 0.680 |
| F12: Inventory Monitor Service | 6% | 10.0 | 0.600 |
| F13: Email Utility | 3% | 10.0 | 0.300 |
| F14: Background Tasks | 3% | 5.0 | 0.150 |
| F15: Frontend Dashboard | 7% | 10.0 | 0.700 |
| F16: Infrastructure (Docker/Nginx) | 5% | 8.5 | 0.425 |
| F17: DB Init Script | 4% | 10.0 | 0.400 |
| F18: Testing Suite | 12% | 5.2 | 0.624 |
| F19: Security Implementation | 8% | 6.75 | 0.540 |
| F20: Business Logic Correctness | 7% | 8.0 | 0.560 |
| **TOTAL** | **140%** | | **11.269** |

**Note:** Rubric weights sum to 140%. Normalized: 11.269 / 1.40 = **8.76/10**

---

## Final Score

```
Weighted Average: 8.76 / 10
Grade: A (8.0 - 8.9)
```

### Grade Bands
| Weighted Average | Grade |
|-----------------|-------|
| 9.0 - 10.0 | A+ |
| **8.0 - 8.9** | **A** |
| 7.0 - 7.9 | B+ |
| 6.0 - 6.9 | B |
| 5.0 - 5.9 | C |
| 4.0 - 4.9 | D |
| Below 4.0 | F |

### Key Strengths
1. **Complete PO workflow** — Full 10-stage forward-only state machine with status history
2. **Strong CRUD coverage** — Products, suppliers, orders, payments, inventory all fully implemented
3. **Rich seed data** — 36 products, 16 POs across all statuses, 90 days of time-series data
4. **Well-designed services** — Audit logging with sensitive field exclusion, inventory monitor with velocity calculation

### Key Weaknesses
1. **Testing quality** (F18: 5.2) — Many tests are empty stubs; others use wrong field names causing failures
2. **Security gaps** (F19: 6.75) — bleach and Fernet in deps but never used; no auth-specific rate limiting
3. **Scheduler not integrated** (F14: 5.0) — Scheduler code exists but never started in FastAPI lifespan
4. **Missing Referrer-Policy** — Backend SecurityHeadersMiddleware omits this required header
