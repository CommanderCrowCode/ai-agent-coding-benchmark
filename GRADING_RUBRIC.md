# Grading Rubric: Small Business Inventory Management System

## Scoring Methodology

### Cross-Team Comparison Rule
Teams may complete different amounts of work. **Only features attempted by ALL teams under comparison should be scored.** If Team A completed Features 1-8 and Team B completed Features 1-5, compare only on Features 1-5. This ensures fair comparison regardless of time allocated.

### Per-Feature Scoring
Each feature module is scored independently on a **0-10 scale** across multiple dimensions. The final score for a feature is the weighted average of its dimension scores.

### Dimension Weights (per feature)
| Dimension | Weight | Description |
|-----------|--------|-------------|
| **Completeness** | 30% | All specified endpoints, fields, and behaviors are present |
| **Correctness** | 30% | Business logic works as specified; edge cases handled |
| **Code Quality** | 20% | Clean, readable, follows project conventions, no anti-patterns |
| **Testing** | 20% | Adequate tests exist and pass for this feature |

### Score Levels
| Score | Label | Meaning |
|-------|-------|---------|
| 10 | Exemplary | Exceeds spec; robust, elegant, well-tested |
| 8-9 | Strong | Fully meets spec with minor polish gaps |
| 6-7 | Adequate | Core functionality works; some gaps or rough edges |
| 4-5 | Partial | Significant portions missing or broken |
| 2-3 | Minimal | Skeleton exists but largely non-functional |
| 0-1 | Absent/Broken | Not implemented or completely non-functional |

---

## Feature Modules

---

### F01: Project Structure & Setup (Weight: 5%)

**What to evaluate:** Directory layout, dependency management, configuration files.

| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| Directory structure matches spec | /2 | 2=exact match, 1=close but missing dirs, 0=significantly different |
| `pyproject.toml` with all dependencies | /2 | 2=all deps listed with correct versions, 1=most present, 0=missing or pip-based |
| `alembic.ini` + alembic directory exists | /1 | 1=present and configured, 0=missing |
| `uv` used for package management | /1 | 1=uv.lock present or evidence of uv usage, 0=pip/other |
| Frontend `package.json` with correct deps | /1 | 1=Next.js 14, React 18, TailwindCSS present, 0=missing |
| `tsconfig.json` and `tailwind.config.ts` | /1 | 1=both present and correct, 0=missing |
| `.env` example or config documentation | /1 | 1=present, 0=missing |
| Clean separation of concerns | /1 | 1=routers/schemas/services/utils separated, 0=monolithic |

**Total: /10**

---

### F02: Database Schema & Models (Weight: 10%)

**What to evaluate:** SQLAlchemy models covering all 20+ tables with correct relationships, types, and constraints.

#### F02a: Auth & Security Tables
| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| `users` table — all fields (email, password_hash, mfa_secret, mfa_enabled, role, is_active) | /2 | 2=all fields, 1=missing 1-2, 0=missing 3+ |
| `audit_log` table — all fields including JSON old/new values | /2 | 2=all fields, 1=missing key fields, 0=missing or wrong |

#### F02b: Products & Suppliers Tables
| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| `suppliers` — all fields (name, legal_name, website, address, payment_terms, lead_time_days, moq, notes) | /1 | 1=complete, 0.5=most fields, 0=missing |
| `supplier_contacts` — FK to supplier, platform enum | /1 | 1=correct with FK, 0=missing |
| `products` — internal_sku (LUMI-xxxx unique), supplier_sku, age ranges, price_usd, units_per_carton, is_best_seller, is_active, supplier FK | /2 | 2=all fields with constraints, 1=most fields, 0=incomplete |
| `price_history` — product_id FK, supplier_id FK, price_usd, effective_date | /1 | 1=correct, 0=missing |

#### F02c: Purchase Orders Tables
| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| `purchase_orders` — po_number (PO-YYYY-NNN unique), status enum, financial fields | /2 | 2=all fields with constraints, 1=partial, 0=missing |
| `po_line_items` — FK to po and product, quantity, unit_price | /1 | 1=correct, 0=missing |
| `po_status_history` — FK to po, old/new status, changed_by, timestamp | /1 | 1=correct, 0=missing |

#### F02d: Shipments Tables
| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| `shipments` — FK to po, carrier, tracking, bill_of_lading, ports, status | /1 | 1=complete, 0=missing |
| `shipment_milestones` — FK to shipment, milestone_type enum, expected/actual dates | /1 | 1=correct with enum, 0=missing |

#### F02e: Payments Table
| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| `payments` — payment_id (PAY-YYYY-NNN unique), FK to po and supplier, Wise fields, status enum, financial fields | /2 | 2=all fields, 1=partial, 0=missing |

#### F02f: Inventory Tables
| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| `inventory` — product_id (unique FK), quantity, reorder_point (default 10), safety_stock_days (default 14), last_counted_at | /2 | 2=all fields with defaults, 1=missing defaults, 0=incomplete |
| `stock_movements` — movement_type enum (IN/OUT/ADJUSTMENT), reference_type enum, reference_id | /1 | 1=correct with enums, 0=missing |
| `sales` — product_id FK, platform enum (SHOPEE/LAZADA/DIRECT), order_reference | /1 | 1=correct, 0=missing |

#### F02g: Planning Tables
| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| `seasonality_factors` — month (1-12), multiplier | /1 | 1=correct, 0=missing |
| `holidays` — country, name, dates, type enum | /1 | 1=correct, 0=missing |
| `settings` — key/value store | /0.5 | 0.5=present, 0=missing |
| `notifications` — product_id FK, alert_type, message, status enum | /0.5 | 0.5=correct, 0=missing |

#### F02h: Indexes & Constraints
| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| All specified indexes created (11 total) | /3 | 3=all present, 2=8-10, 1=4-7, 0=<4 |
| Unique constraints on internal_sku, po_number, payment_id | /2 | 2=all three, 1=some, 0=none |
| Foreign key relationships correctly defined | /2 | 2=all correct, 1=most, 0=many missing |
| Proper use of enums for status fields | /1 | 1=enums used, 0=plain strings |
| Table count >= 20 | /2 | 2=20+, 1=15-19, 0=<15 |

**Subtotal: /30, normalize to /10**

---

### F03: Application Setup & Configuration (Weight: 5%)

**What to evaluate:** `app/main.py` and `app/config.py`

| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| FastAPI app with lifespan manager | /1 | 1=lifespan used for background tasks, 0=missing |
| CORS configured for specified origins | /1 | 1=both origins whitelisted, 0.5=one only, 0=wide open or missing |
| Global rate limiter: 100 req/min | /1 | 1=implemented with slowapi, 0=missing |
| Auth endpoint rate limit: 5 req/min | /1 | 1=specific limit on auth routes, 0=missing |
| Security headers middleware (all 4 headers) | /2 | 2=all 4 headers, 1=2-3 headers, 0=0-1 headers |
| Health check `GET /health` returns `{"status": "ok"}` | /1 | 1=works, 0=missing |
| Pydantic Settings with all env vars | /2 | 2=all vars listed, 1=most, 0=incomplete |
| `app/database.py` — proper session management, `get_db` dependency | /1 | 1=correct pattern, 0=missing |

**Total: /10**

---

### F04: Auth Router (Weight: 8%)

**What to evaluate:** `routers/auth.py` — registration, login, JWT, MFA

| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| `POST /api/auth/register` — creates user, hashes password with bcrypt | /2 | 2=works with bcrypt (work factor 12), 1=works but wrong hashing, 0=broken |
| `POST /api/auth/login` — verifies credentials, returns JWT access + refresh tokens | /2 | 2=both tokens returned correctly, 1=only access token, 0=broken |
| `POST /api/auth/refresh` — exchanges refresh token for new access token | /1 | 1=works, 0=missing |
| `POST /api/auth/mfa/setup` — generates TOTP secret, returns QR URI | /1.5 | 1.5=TOTP with QR URI, 1=TOTP without QR, 0=missing |
| `POST /api/auth/mfa/verify` — validates TOTP code, enables MFA | /1.5 | 1.5=validates and enables, 1=partial, 0=missing |
| JWT uses HS256 with 15-min access token expiry | /1 | 1=correct algo and expiry, 0.5=one correct, 0=neither |
| Refresh token: 7-day expiry | /0.5 | 0.5=correct, 0=missing |
| Rate limiting: 5 req/min on auth endpoints | /0.5 | 0.5=implemented, 0=missing |

**Total: /10**

---

### F05: Products Router (Weight: 7%)

**What to evaluate:** `routers/products.py` — CRUD with price history

| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| `GET /api/products` — list with filtering (best_seller, active) | /2 | 2=both filters work, 1=one filter, 0=no filtering |
| `GET /api/products/{id}` — detail with supplier info | /1 | 1=includes supplier relationship, 0=missing |
| `POST /api/products` — create product, record initial price in price_history | /2 | 2=creates + records price, 1=creates only, 0=broken |
| `PUT /api/products/{id}` — update; records price_history if price changed | /2 | 2=detects price change + records, 1=updates but no price history, 0=broken |
| `DELETE /api/products/{id}` — soft delete (is_active=False) | /1.5 | 1.5=soft delete, 0=hard delete or missing |
| Proper Pydantic schemas for request/response | /1 | 1=well-defined schemas, 0.5=partial, 0=no schemas |
| Input validation (e.g., valid SKU format) | /0.5 | 0.5=validates, 0=no validation |

**Total: /10**

---

### F06: Suppliers Router (Weight: 5%)

**What to evaluate:** `routers/suppliers.py` — CRUD with contacts

| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| `GET /api/suppliers` — list all | /1.5 | 1.5=works, 0=missing |
| `GET /api/suppliers/{id}` — detail with contacts | /2 | 2=includes contacts, 1=detail without contacts, 0=missing |
| `GET /api/suppliers/{id}/contacts` — contacts only | /1.5 | 1.5=works, 0=missing |
| `POST /api/suppliers` — create | /2 | 2=works with validation, 1=works no validation, 0=missing |
| `PUT /api/suppliers/{id}` — update | /2 | 2=works with validation, 1=works no validation, 0=missing |
| Proper Pydantic schemas | /1 | 1=well-defined, 0=missing |

**Total: /10**

---

### F07: Purchase Orders Router (Weight: 10%)

**What to evaluate:** `routers/orders.py` — CRUD, status workflow, line items

| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| `GET /api/orders` — list with filters (status, supplier_id) | /1 | 1=both filters, 0.5=one filter, 0=no filtering |
| `GET /api/orders/{id}` — detail with line_items AND status_history | /1.5 | 1.5=both included, 1=one included, 0=neither |
| `GET /api/orders/workflow-stats` — count per status | /1 | 1=works, 0=missing |
| `POST /api/orders` — create with line items | /1 | 1=creates order + line items, 0=missing |
| Auto-generate `PO-YYYY-NNN` format | /1 | 1=correct format + auto-increment, 0.5=format but not auto, 0=missing |
| Calculate totals from line items (subtotal = sum of qty * unit_price) | /1 | 1=auto-calculated, 0=manual only |
| `PUT /api/orders/{id}` — update notes, expected_ready_date | /0.5 | 0.5=works, 0=missing |
| `POST /api/orders/{id}/status` — advance status | /1 | 1=works, 0=missing |
| **Status workflow: all 10 stages defined** (DRAFT->SENT->CONFIRMED->IN_PRODUCTION->SHIPPED->IN_TRANSIT->CUSTOMS->RECEIVED->QCD->STOCKED) | /1 | 1=all 10, 0.5=most, 0=few |
| **Forward-only status transitions enforced** (cannot go backwards) | /1 | 1=enforced with error on invalid transition, 0=allows any transition |
| Status change recorded in `po_status_history` | /0.5 | 0.5=recorded, 0=not recorded |

**Total: /10 (sum=10)**

---

### F08: Payments Router (Weight: 7%)

**What to evaluate:** `routers/payments.py` — CRUD, summary, Wise stub

| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| `GET /api/payments` — list with status filter | /1 | 1=filter works, 0.5=list only, 0=missing |
| `GET /api/payments/{id}` — detail | /1 | 1=works, 0=missing |
| `GET /api/payments/summary` — monthly totals, YTD, total fees | /2 | 2=all three aggregations, 1=partial, 0=missing |
| `POST /api/payments` — create with auto-generated PAY-YYYY-NNN | /2 | 2=correct format + auto-increment, 1=creates but wrong format, 0=missing |
| `PUT /api/payments/{id}` — update status/notes, logs to audit_log | /2 | 2=updates + audit logged, 1=updates only, 0=missing |
| `GET /api/payments/{id}/sync-wise` — stub returns current data | /1 | 1=stub exists, 0=missing |
| Proper Pydantic schemas | /1 | 1=well-defined, 0=missing |

**Total: /10**

---

### F09: Inventory Router (Weight: 15%)

**What to evaluate:** `routers/inventory.py` — the most complex module

#### F09a: Read Endpoints
| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| `GET /api/inventory/` — list all with stock status (OK/LOW/OUT_OF_STOCK) | /2 | 2=correct status calculation, 1=lists but no status, 0=missing |
| Status filter query param (`?status_filter=LOW`) | /1 | 1=works, 0=missing |
| Uses `joinedload` for product relationship | /0.5 | 0.5=uses eager loading, 0=N+1 queries |
| `GET /api/inventory/stats` — aggregate statistics | /1.5 | 1.5=all 5 stats (total_value, total_products, in/low/out counts), 1=partial, 0=missing |
| `GET /api/inventory/{product_id}` — detail; auto-creates if missing | /1 | 1=auto-creates with qty=0, 0.5=detail without auto-create, 0=missing |
| `GET /api/inventory/{product_id}/movements` — paginated history | /1 | 1=paginated, 0.5=unpaginated, 0=missing |

#### F09b: Receive Shipment
| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| `POST /api/inventory/receive-shipment` — accepts bulk items + reference | /1 | 1=correct request body, 0=missing |
| Validates ALL products exist before processing (atomic) | /1.5 | 1.5=pre-validates atomically, 0.5=processes one-by-one, 0=no validation |
| Creates `IN` stock movement per item | /1 | 1=movements created, 0=missing |
| Updates inventory quantity and `updated_at` | /0.5 | 0.5=both updated, 0=missing |

#### F09c: Sales
| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| `POST /api/inventory/sale` — validates sufficient stock | /1 | 1=validates, 0=no validation |
| Creates `OUT` movement + sale record (platform, order_reference) | /1 | 1=both created, 0.5=one only, 0=neither |
| Decrements inventory | /0.5 | 0.5=correct, 0=missing |
| `POST /api/inventory/bulk-sale` — atomic bulk sales | /1.5 | 1.5=atomic with pre-validation, 1=works but not atomic, 0=missing |
| Insufficient stock returns 400 error | /1 | 1=correct error code, 0=wrong code or no error |

#### F09d: Adjustment
| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| `POST /api/inventory/adjust` — accepts absolute target quantity | /1 | 1=absolute (not delta), 0.5=delta-based, 0=missing |
| Prevents negative inventory | /1 | 1=validates, 0=allows negative |
| Creates `ADJUSTMENT` movement with correct delta | /1 | 1=delta calculated correctly, 0=wrong or missing |
| Updates `last_counted_at` | /0.5 | 0.5=updated, 0=missing |

**Subtotal: /18, normalize to /10**

---

### F10: Dashboard Router (Weight: 5%)

**What to evaluate:** `routers/dashboard.py` — KPIs and summaries

| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| `GET /api/dashboard/` — returns all 4 KPIs | /4 | 1 per KPI: total_inventory_value, pending_orders, low_stock_alerts, mtd_revenue |
| `GET /api/dashboard/order-pipeline` — orders grouped by status | /3 | 3=correct grouping, 1=partial, 0=missing |
| `GET /api/dashboard/inventory-health` — per-SKU status summary | /3 | 3=correct per-SKU data, 1=partial, 0=missing |

**Total: /10**

---

### F11: Audit Service (Weight: 8%)

**What to evaluate:** `services/audit.py` — change tracking with sensitive field exclusion

| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| `log_action` function exists with correct signature | /1 | 1=all params (db, user_id, table_name, record_id, action, old_values, new_values, request), 0=missing |
| Logs CREATE actions with new_values | /1 | 1=correct, 0=missing |
| Logs UPDATE actions with old AND new values | /1.5 | 1.5=both, 1=one only, 0=missing |
| Logs DELETE actions | /1 | 1=correct, 0=missing |
| Excludes `password_hash` and `mfa_secret` from logged values | /1.5 | 1.5=both excluded, 1=one excluded, 0=neither |
| Only logs when values actually change (change detection) | /1.5 | 1.5=skips no-op updates, 0=logs everything |
| Captures IP address from `request.client.host` | /0.5 | 0.5=captured, 0=missing |
| Captures User-Agent from headers | /0.5 | 0.5=captured, 0=missing |
| Audit logging integrated into all mutation endpoints | /1 | 1=all CRUD endpoints call audit, 0.5=some, 0=none |

**Total: /10**

---

### F12: Inventory Monitor Service (Weight: 6%)

**What to evaluate:** `services/inventory_monitor.py` — stockout prediction and alerting

| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| `_calculate_days_until_stockout` — queries last 90 days of sales | /2 | 2=correct query + calculation, 1=calculation exists but wrong window, 0=missing |
| `daily_velocity = total_sold / 90` formula correct | /1 | 1=correct, 0=wrong |
| Returns infinity (or equivalent) when no sales | /1 | 1=handles zero velocity, 0=division by zero |
| `_should_send_alert` — checks notifications table for 24h dedup | /2 | 2=correct 24h window check, 1=checks but wrong window, 0=missing |
| `check_low_stock_and_alert` — iterates LOW/OUT_OF_STOCK products | /2 | 2=correct filtering + iteration, 1=partial, 0=missing |
| Creates notification record when alert sent | /1 | 1=recorded, 0=missing |
| Integrates with email sending | /1 | 1=calls email utility, 0=missing |

**Total: /10**

---

### F13: Email Utility (Weight: 3%)

**What to evaluate:** `utils/email.py` — low stock alert emails

| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| `send_low_stock_alert` function with correct params | /2 | 2=all params (sku, qty, reorder_point, days_until_stockout, recipients), 0=missing |
| HTML email with styled alert card | /3 | 3=well-formatted HTML, 2=basic HTML, 1=plain text, 0=missing |
| Includes: SKU, current qty, reorder point, days until stockout | /2 | 0.5 per field included |
| Action buttons (Create PO, View Inventory) | /1 | 1=buttons present, 0=missing |
| Sends via SMTP | /2 | 2=uses SMTP config from settings, 1=hardcoded, 0=missing |

**Total: /10**

---

### F14: Background Tasks / Scheduler (Weight: 3%)

**What to evaluate:** `tasks/scheduler.py` — periodic inventory monitoring

| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| Runs `check_low_stock_and_alert()` periodically | /4 | 4=every hour via asyncio, 2=runs but wrong interval, 0=missing |
| Integrated with FastAPI lifespan | /3 | 3=proper lifespan integration, 1=other mechanism, 0=missing |
| Error handling (doesn't crash on failure) | /2 | 2=catches exceptions + logs, 1=basic try/catch, 0=no handling |
| Graceful shutdown | /1 | 1=cancels task on shutdown, 0=no cleanup |

**Total: /10**

---

### F15: Frontend Dashboard (Weight: 7%)

**What to evaluate:** Next.js 14 dashboard page

| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| Client component (`'use client'`) | /0.5 | 0.5=present, 0=missing |
| Fetches `GET /api/dashboard/kpis` on mount | /1.5 | 1.5=correct fetch on mount, 0=missing |
| 4 KPI cards displayed (Inventory Value, Pending Orders, Low Stock, MTD Revenue) | /2 | 0.5 per card |
| Responsive grid (1->2->4 columns) | /1 | 1=responsive breakpoints, 0.5=fixed layout, 0=missing |
| Loading spinner state | /1 | 1=shows during fetch, 0=missing |
| Error fallback with zeroes | /1 | 1=graceful fallback, 0=crashes on error |
| Order Pipeline section (even if "Coming soon") | /0.5 | 0.5=section exists, 0=missing |
| Inventory Health section (even if "Coming soon") | /0.5 | 0.5=section exists, 0=missing |
| TailwindCSS styling throughout | /1 | 1=consistent Tailwind usage, 0.5=mixed, 0=no Tailwind |
| Light theme (white bg, dark text, correct color palette) | /0.5 | 0.5=matches spec, 0=dark theme or unstyled |
| `layout.tsx` and `globals.css` present | /0.5 | 0.5=both present, 0=missing |

**Total: /10**

---

### F16: Infrastructure — Docker & Nginx (Weight: 5%)

**What to evaluate:** `docker-compose.yml` and `nginx/nginx.conf`

| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| 3 services defined (frontend, backend, nginx) | /2 | 2=all three, 1=two, 0=one or none |
| Frontend: port 3000, resource limits (0.5 CPU / 512MB) | /1 | 1=correct, 0.5=port only, 0=missing |
| Backend: port 8000, resource limits (0.75 CPU / 1GB), health check every 30s | /1.5 | 1.5=all three, 1=port+limits, 0.5=port only, 0=missing |
| Nginx: ports 80 + 443, SSL termination | /1 | 1=both ports + SSL, 0.5=one port, 0=missing |
| Custom bridge network | /0.5 | 0.5=defined, 0=missing |
| Backend reads from `.env` file | /0.5 | 0.5=env_file configured, 0=missing |
| Nginx proxies `/api/` to backend:8000 | /1 | 1=correct proxy, 0=missing |
| Nginx proxies `/` to frontend:3000 | /1 | 1=correct proxy, 0=missing |
| SSL certs in `./certs/` referenced | /0.5 | 0.5=configured, 0=missing |
| Compose file is valid and well-structured | /1 | 1=valid YAML, parseable, 0=syntax errors |

**Total: /10**

---

### F17: Database Initialization Script (Weight: 4%)

**What to evaluate:** `webapp/backend/init_db.py` — seed data

| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| Admin user: admin@example.com / admin123 with bcrypt hash | /2 | 2=correct email + properly hashed, 1=user but wrong hash, 0=missing |
| One supplier with contacts | /1.5 | 1.5=supplier + contacts, 1=supplier only, 0=missing |
| 4-8 sample products | /1.5 | 1.5=4-8 products with proper fields, 1=1-3 products, 0=missing |
| 2 purchase orders (one CONFIRMED, one DRAFT) with line items | /2 | 2=both with correct statuses + line items, 1=orders without line items, 0=missing |
| Inventory records (one healthy, one low stock) | /1.5 | 1.5=both scenarios, 1=only one, 0=missing |
| Sample stock movements (IN and OUT) | /1.5 | 1.5=both types, 1=one type, 0=missing |

**Total: /10**

---

### F18: Testing Suite (Weight: 12%)

**What to evaluate:** All test files, coverage, and infrastructure

#### F18a: Test Infrastructure
| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| In-memory SQLite for test isolation | /2 | 2=in-memory DB per test/session, 1=file-based SQLite, 0=no test DB |
| Autouse fixtures for setup/teardown | /1 | 1=proper fixtures, 0=manual setup |
| Override `get_db()` dependency per test | /1.5 | 1.5=correct FastAPI dependency override, 0=no override |
| pytest-asyncio for async tests | /0.5 | 0.5=configured, 0=missing |

#### F18b: Test Coverage by Module
Score each test file on: (a) exists, (b) test count meets target, (c) tests pass, (d) covers specified scenarios.

| Test File | Target Count | Points | Scoring Guide |
|-----------|-------------|--------|---------------|
| `test_products.py` | 10 | /1 | 1=10+ passing, 0.5=5-9, 0=<5 |
| `test_orders.py` | 9 | /1 | 1=9+ passing, 0.5=5-8, 0=<5 |
| `test_payments.py` | 11 | /1 | 1=11+ passing, 0.5=6-10, 0=<6 |
| `test_inventory.py` | 22 | /1.5 | 1.5=22+ passing, 1=15-21, 0.5=8-14, 0=<8 |
| `test_suppliers.py` | 10 | /1 | 1=10+ passing, 0.5=5-9, 0=<5 |
| `test_auth.py` | 19 | /1.5 | 1.5=19+ passing, 1=12-18, 0.5=6-11, 0=<6 |
| `test_audit.py` | 18 | /1.5 | 1.5=18+ passing, 1=12-17, 0.5=6-11, 0=<6 |
| `test_inventory_monitor.py` | 18 | /1.5 | 1.5=18+ passing, 1=12-17, 0.5=6-11, 0=<6 |

#### F18c: Test Quality
| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| Total test count >= 117 | /2 | 2=117+, 1=80-116, 0.5=40-79, 0=<40 |
| All tests pass (0 failures) | /3 | 3=100% pass, 2=95%+, 1=80%+, 0=<80% |
| Tests cover edge cases (not just happy path) | /2 | 2=good edge case coverage, 1=some, 0=happy path only |
| Tests are independent (no order dependency) | /1 | 1=independent, 0=order-dependent |

**Subtotal: /22, normalize to /10**

---

### F19: Security Implementation (Weight: 8%)

**What to evaluate:** Cross-cutting security requirements

| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| Passwords: bcrypt with work factor 12 | /1.5 | 1.5=bcrypt + rounds=12, 1=bcrypt default rounds, 0=other hashing |
| JWT: HS256 algorithm | /0.5 | 0.5=HS256, 0=other |
| JWT: 15-min access tokens | /0.5 | 0.5=correct, 0=wrong expiry |
| JWT: 7-day refresh tokens | /0.5 | 0.5=correct, 0=wrong expiry |
| MFA: TOTP via pyotp | /1 | 1=pyotp used, 0=missing |
| Rate limiting: 100/min global | /0.5 | 0.5=implemented, 0=missing |
| Rate limiting: 5/min on auth | /0.5 | 0.5=implemented, 0=missing |
| Input sanitization: bleach for HTML | /1 | 1=bleach used on inputs, 0=no sanitization |
| Encrypted storage: Fernet for sensitive fields | /1.5 | 1.5=Fernet encryption for bank details, 1=some encryption, 0=plaintext |
| No raw SQL with user input (ORM only) | /1 | 1=ORM throughout, 0=raw SQL with user input |
| CORS: whitelist only (not wildcard) | /0.5 | 0.5=whitelisted, 0=wildcard |
| Security headers (X-Content-Type-Options, X-Frame-Options, Referrer-Policy, CSP) | /1 | 0.25 per header |

**Total: /10**

---

### F20: Business Logic Correctness (Weight: 7%)

**What to evaluate:** Cross-cutting business rules

| Criterion | Points | Scoring Guide |
|-----------|--------|---------------|
| **Inventory Status Rules** correct (OUT_OF_STOCK: qty==0, LOW: 0<qty<=reorder_point, OK: qty>reorder_point) | /2 | 2=all three correct, 1=two correct, 0=wrong logic |
| **Days Until Stockout** formula (daily_velocity = sold_90d/90, days = qty/velocity) | /1.5 | 1.5=correct formula, 1=formula exists but wrong, 0=missing |
| **Alert Deduplication** — one email per product per 24h | /1.5 | 1.5=correct dedup, 0=no dedup |
| **PO Number Generation** — `PO-YYYY-NNN` with auto-increment | /1 | 1=correct pattern, 0.5=wrong format, 0=missing |
| **Payment ID Generation** — `PAY-YYYY-NNN` with auto-increment | /1 | 1=correct pattern, 0.5=wrong format, 0=missing |
| **Atomic Bulk Operations** — receive-shipment and bulk-sale validate all before modifying any | /2 | 2=both atomic, 1=one atomic, 0=neither |
| **Database transactions** — proper rollback on failure | /1 | 1=transactions used, 0=no transaction management |

**Total: /10**

---

## Summary Score Sheet

Use this table to record scores for each team. Only fill in features that the team attempted.

| Feature | Weight | Team A | Team B | Team C | Notes |
|---------|--------|--------|--------|--------|-------|
| F01: Project Structure & Setup | 5% | /10 | /10 | /10 | |
| F02: Database Schema & Models | 10% | /10 | /10 | /10 | |
| F03: App Setup & Configuration | 5% | /10 | /10 | /10 | |
| F04: Auth Router | 8% | /10 | /10 | /10 | |
| F05: Products Router | 7% | /10 | /10 | /10 | |
| F06: Suppliers Router | 5% | /10 | /10 | /10 | |
| F07: Purchase Orders Router | 10% | /10 | /10 | /10 | |
| F08: Payments Router | 7% | /10 | /10 | /10 | |
| F09: Inventory Router | 15% | /10 | /10 | /10 | |
| F10: Dashboard Router | 5% | /10 | /10 | /10 | |
| F11: Audit Service | 8% | /10 | /10 | /10 | |
| F12: Inventory Monitor Service | 6% | /10 | /10 | /10 | |
| F13: Email Utility | 3% | /10 | /10 | /10 | |
| F14: Background Tasks | 3% | /10 | /10 | /10 | |
| F15: Frontend Dashboard | 7% | /10 | /10 | /10 | |
| F16: Infrastructure (Docker/Nginx) | 5% | /10 | /10 | /10 | |
| F17: DB Init Script | 4% | /10 | /10 | /10 | |
| F18: Testing Suite | 12% | /10 | /10 | /10 | |
| F19: Security Implementation | 8% | /10 | /10 | /10 | |
| F20: Business Logic Correctness | 7% | /10 | /10 | /10 | |
| **Weighted Total** | **100%** | | | | |

### Calculating Cross-Team Scores

1. Identify which features ALL teams attempted (the "common set")
2. For each team, calculate the weighted score using ONLY the common set features
3. Re-normalize weights to sum to 100% across the common set
4. Formula: `Normalized Score = (Sum of weighted feature scores for common set) / (Sum of weights for common set) * 100`

**Example:**
- Common features: F01, F02, F03, F04, F05 (weights: 5+10+5+8+7 = 35%)
- Team A raw weighted sum for these: 8.2
- Team A normalized: 8.2 / 35 * 100 = 23.4 → then scale to /10: 8.2/3.5 = **2.34/3.5** or simply report the weighted average directly

### Final Grade Bands

| Weighted Average | Grade | Interpretation |
|-----------------|-------|----------------|
| 9.0 - 10.0 | A+ | Production-ready quality |
| 8.0 - 8.9 | A | Excellent, minor polish needed |
| 7.0 - 7.9 | B+ | Strong, some gaps |
| 6.0 - 6.9 | B | Solid foundation, noticeable gaps |
| 5.0 - 5.9 | C | Functional but significant issues |
| 4.0 - 4.9 | D | Major pieces missing or broken |
| Below 4.0 | F | Insufficient |

---

## Appendix A: Quick Verification Commands

These commands can help verify specific criteria quickly:

```bash
# Count total tables in models.py
grep -c "class.*Base" webapp/backend/app/models.py

# Count total tests
cd webapp/backend && uv run pytest --collect-only -q 2>/dev/null | tail -1

# Run all tests
cd webapp/backend && uv run pytest -v

# Check health endpoint
curl -s http://localhost:8000/health | python3 -m json.tool

# Check rate limiting (should get 429 after 5 rapid requests)
for i in {1..6}; do curl -s -o /dev/null -w "%{http_code}\n" -X POST http://localhost:8000/api/auth/login -H "Content-Type: application/json" -d '{"email":"test@test.com","password":"wrong"}'; done

# Verify Docker services
docker compose -f webapp/infrastructure/docker-compose.yml config --quiet && echo "Valid" || echo "Invalid"

# Check index count in models
grep -c "Index(" webapp/backend/app/models.py

# Verify security headers
curl -sI http://localhost:8000/health | grep -E "(X-Content-Type|X-Frame|Referrer-Policy|Content-Security)"

# Verify CORS not wildcard
grep -r "allow_origins" webapp/backend/app/main.py

# Check for raw SQL (should return nothing)
grep -rn "text(" webapp/backend/app/routers/ | grep -v "# " | head -5

# Verify bcrypt rounds
grep -r "rounds\|work_factor\|CryptContext" webapp/backend/app/ | head -5
```

## Appendix B: Acceptance Criteria Mapping

Maps the 10 acceptance criteria from the spec to feature modules:

| # | Acceptance Criterion | Primary Feature | How to Verify |
|---|---------------------|-----------------|---------------|
| 1 | All 117+ pytest tests pass | F18 | `uv run pytest` shows 117+ passed, 0 failed |
| 2 | `GET /health` returns 200 | F03 | `curl localhost:8000/health` |
| 3 | Can create supplier->product->PO->advance all 10 stages | F06, F05, F07 | Run through full workflow via API |
| 4 | Receive shipment updates inventory | F09 | POST receive-shipment, check GET inventory |
| 5 | Sale beyond stock returns 400 | F09 | POST sale with qty > stock |
| 6 | Low stock in filtered list | F09 | GET /api/inventory/?status_filter=LOW |
| 7 | All mutations in audit_log | F11 | Check audit_log after any CREATE/UPDATE/DELETE |
| 8 | Auth rate limiting returns 429 | F04, F03 | 6 rapid login attempts |
| 9 | Docker Compose starts all 3 services | F16 | `docker compose up -d` + health checks |
| 10 | Frontend dashboard displays KPI cards | F15 | Open browser to localhost:3000 |
