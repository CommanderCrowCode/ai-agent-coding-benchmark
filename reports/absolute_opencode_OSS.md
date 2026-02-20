# Absolute Grading Report: OpenCode OSS

## Summary
- **Total Score: 38/200**
- **Grade: F (Fundamentally incomplete)**

> **Overview:** The project delivers infrastructure configuration (Docker/Nginx) at production quality and demonstrates strong knowledge of auth patterns (JWT, MFA, bcrypt). However, the backend cannot start due to missing `db.py` module (import crash), only 2 of 20+ database models exist, the entire inventory and dashboard modules are 501 stubs, the frontend has a JSX syntax error preventing compilation, and no meaningful tests exist. The project is a partially-scaffolded skeleton that cannot run.

---

## Section Scores

### Section 1: Does It Run? (30 points)

| # | Criterion | Max | Score | Evidence |
|---|-----------|-----|-------|---------|
| 1.1 | Backend starts without crashing | 5 | 0 | `app/main.py:3` → `from .routers import orders` → `routers/__init__.py:1` → `from . import orders` → `orders.py:8` → `from ..db import get_db` → **ModuleNotFoundError**: no `app.db` module. Also `main.py:4` → `from .tasks.scheduler import start_scheduler, Request, Response` → **ImportError**: `scheduler.py` doesn't export these names. The import chain crashes before the FastAPI app is created. |
| 1.2 | `GET /health` returns `{"status": "ok"}` with HTTP 200 | 3 | 0 | Backend cannot start, so endpoint is unreachable. Code exists at `app/main.py:57-59`. |
| 1.3 | Frontend starts without crashing | 5 | 1 | `package.json` has correct deps (Next.js 14, React 18, Tailwind). However, `page.tsx:59` has JSX syntax error: `className={...`)}>` — extra `)`. Also **`layout.tsx` is missing** (required by Next.js App Router). Would fail to compile. |
| 1.4 | Frontend page loads in browser | 3 | 0 | Cannot compile due to syntax error in `page.tsx:59` |
| 1.5 | Database tables get created | 5 | 0 | No `database.py` or `db.py` exists. No migration scripts. No auto-create logic. Only 2 model files exist (`models/audit_log.py`, `models/notification.py`) with no `models/__init__.py` or `models/base.py` to tie them together. |
| 1.6 | `init_db.py` runs and seeds data | 4 | 0 | **No `init_db.py` file exists** anywhere in the project |
| 1.7 | Docker Compose valid YAML, 3 services | 3 | 3 | Valid YAML. Three services: `frontend` (node:18-alpine), `backend` (python:3.11-slim), `nginx` (nginx:stable-alpine). Clean structure with `depends_on`, networks. (`infrastructure/docker-compose.yml`) |
| 1.8 | Nginx config parses without error | 2 | 2 | Clean nginx.conf with proper `upstream` blocks, `server` block, SSL config, proxy rules. Standard syntax. (`infrastructure/nginx/nginx.conf`) |

**Section Total: 6/30**

---

### Section 2: Test Suite (35 points)

| # | Criterion | Max | Score | Evidence |
|---|-----------|-----|-------|---------|
| 2.1 | Test suite runs at all | 3 | 0 | Only `tests/test_health.py` exists. It references a `client` fixture (`test_health.py:1`) but **no `conftest.py` exists** to define it. `uv run pytest` would fail with fixture not found error. |
| 2.2 | Number of tests collected | 10 | 0 | 1 test defined. Even if collected, 0-29 → 0 points. |
| 2.3 | Pass rate | 12 | 0 | The single test would fail (missing `client` fixture). Pass rate = 0%. <50% → 0 points. |
| 2.4 | All 8 test files exist | 4 | 0 | Only `test_health.py` present. Missing all 8 required files: `test_products`, `test_orders`, `test_payments`, `test_inventory`, `test_suppliers`, `test_auth`, `test_audit`, `test_inventory_monitor`. 0 of 8 × 0.5 = 0. |
| 2.5 | Test isolation | 3 | 0 | No meaningful tests to evaluate isolation |
| 2.6 | In-memory SQLite with proper `get_db` override | 3 | 0 | No `conftest.py`, no test database, no `get_db` override |

**Section Total: 0/35**

---

### Section 3: Schema Completeness (20 points)

| # | Criterion | Max | Score | Evidence |
|---|-----------|-----|-------|---------|
| 3.1 | Table count | 5 | 0 | Only 2 tables defined: `audit_logs` (`models/audit_log.py`) and `notifications` (`models/notification.py`). <10 → 0 points. Missing: users, suppliers, supplier_contacts, products, price_history, purchase_orders, po_line_items, po_status_history, shipments, shipment_milestones, payments, inventory, stock_movements, sales, seasonality_factors, holidays, settings. |
| 3.2 | All enum types properly defined | 3 | 0 | `PO_STATUS_ORDER` list exists in `routers/orders.py:13-24` but is a plain Python list, not a SQLAlchemy `Enum` type. No other enums defined. Need at least 5 distinct enums. |
| 3.3 | All foreign key relationships | 3 | 0.5 | Only `notifications.product_id → products.id` FK exists (`models/notification.py:10`), but the `products` table itself doesn't exist as a model. |
| 3.4 | Unique constraints on internal_sku, po_number, payment_id | 2 | 0 | No `unique=True` or `UniqueConstraint` anywhere in models |
| 3.5 | All 11 specified indexes | 3 | 0 | Only `audit_logs.id` has `index=True` (as primary key). No business indexes defined. |
| 3.6 | Default values (reorder_point=10, safety_stock_days=14) | 1 | 0 | No inventory model exists |
| 3.7 | JSON columns for audit_log old_values/new_values | 1 | 0.5 | `audit_log` has `changes = Column(JSON, nullable=False)` (`models/audit_log.py:13`) — single JSON column instead of separate `old_values`/`new_values` columns |
| 3.8 | Proper timestamps on relevant tables | 2 | 0.5 | `audit_log` has `timestamp` with `default=datetime.utcnow`. `notification` has `created_at`. No `updated_at` on any table. Only 2 tables total. |

**Section Total: 1.5/20**

---

### Section 4: API Endpoints — Do They Exist and Work? (50 points)

> **CRITICAL NOTE:** The backend cannot start due to import errors. No endpoint can actually be reached. Scores below reflect code existence with heavy penalty for non-functionality.

### Auth (8 points)
| # | Criterion | Points | Score | Evidence |
|---|-----------|--------|-------|---------|
| 4.1 | POST /api/auth/register | 2 | 0.5 | Code exists with bcrypt hashing, creates user in `users_db` dict (`auth.py:63-75`). Backend won't start. |
| 4.2 | POST /api/auth/login — access + refresh JWT | 2 | 0.5 | Returns `Token(access_token=..., refresh_token=...)` (`auth.py:78-87`). Backend won't start. |
| 4.3 | POST /api/auth/refresh | 1.5 | 0.5 | Decodes refresh token, generates new pair (`auth.py:90-103`). Backend won't start. |
| 4.4 | POST /api/auth/mfa/setup — TOTP + QR URI | 1.5 | 0.5 | `pyotp.random_base32()` + `provisioning_uri()` (`auth.py:106-119`). Backend won't start. |
| 4.5 | POST /api/auth/mfa/verify | 1 | 0.25 | `totp.verify(code)` + enables MFA (`auth.py:122-131`). Backend won't start. |

**Auth Subtotal: 2.25/8**

### Products (6 points)
| # | Criterion | Points | Score | Evidence |
|---|-----------|--------|-------|---------|
| 4.6 | GET /api/products — filter by best_seller + active | 1.5 | 0.5 | Both filters in code (`products.py:39-48`). In-memory, backend won't start. |
| 4.7 | GET /api/products/{id} — includes supplier | 1 | 0 | Returns product but **no supplier info** (`products.py:52-57`) |
| 4.8 | POST /api/products — creates + price_history | 1.5 | 0.5 | Creates and records price (`products.py:60-67`). In-memory. |
| 4.9 | PUT /api/products/{id} — price_history on change | 1.5 | 0.5 | Detects price change (`products.py:77`). In-memory. |
| 4.10 | DELETE /api/products/{id} — soft delete | 0.5 | 0.25 | `is_active = False` (`products.py:91`). In-memory. |

**Products Subtotal: 1.75/6**

### Suppliers (4 points)
| # | Criterion | Points | Score | Evidence |
|---|-----------|--------|-------|---------|
| 4.11 | GET /api/suppliers — list | 0.5 | 0.25 | In-memory list (`suppliers.py:47-49`). Backend won't start. |
| 4.12 | GET /api/suppliers/{id} — with contacts | 1 | 0.25 | Detail without contacts in response (`suppliers.py:52-57`) |
| 4.13 | GET /api/suppliers/{id}/contacts | 0.5 | 0.25 | Separate contacts endpoint (`suppliers.py:60-69`). Backend won't start. |
| 4.14 | POST /api/suppliers — create | 1 | 0.25 | In-memory creation (`suppliers.py:72-76`). Backend won't start. |
| 4.15 | PUT /api/suppliers/{id} — update | 1 | 0.25 | In-memory update (`suppliers.py:79-87`). Backend won't start. |

**Suppliers Subtotal: 1.25/4**

### Purchase Orders (10 points)
> Router has broken `from ..db import get_db` import AND backend can't start.

| # | Criterion | Points | Score | Evidence |
|---|-----------|--------|-------|---------|
| 4.16 | GET /api/orders — filter by status + supplier_id | 1 | 0.25 | Filter code exists (`orders.py:40-53`) |
| 4.17 | GET /api/orders/{id} — line_items + status_history | 1.5 | 0.25 | Schema has both fields (`schemas/order.py:42-43`) |
| 4.18 | GET /api/orders/workflow-stats | 1 | 0.25 | Group-by query code (`orders.py:65-72`) |
| 4.19 | POST /api/orders — line items, PO#, totals | 3 | 0.5 | PO-YYYY-NNN generation + line items (`orders.py:78-105`). **No total auto-calculation.** |
| 4.20 | PUT /api/orders/{id} — update notes | 0.5 | 0 | Code exists but minimal |
| 4.21 | POST /api/orders/{id}/status — advance + history | 3 | 0.5 | Forward-only validation exists (`orders.py:27-37`). **No status history recording.** |

**Orders Subtotal: 1.75/10**

### Payments (6 points)
> Same broken import issue as orders.

| # | Criterion | Points | Score | Evidence |
|---|-----------|--------|-------|---------|
| 4.22 | GET /api/payments — filter by status | 0.5 | 0.25 | Filter code exists (`payments.py:14-18`) |
| 4.23 | GET /api/payments/{id} — detail | 0.5 | 0.25 | Code exists (`payments.py:21-26`) |
| 4.24 | GET /api/payments/summary — monthly, YTD, fees | 2 | 0.5 | All three aggregations coded (`payments.py:31-48`) |
| 4.25 | POST /api/payments — PAY-YYYY-NNN | 1.5 | 0.5 | Correct format with auto-increment (`payments.py:55-72`) |
| 4.26 | PUT /api/payments/{id} — update + audit log | 1 | 0.25 | Updates and calls `log_action` (`payments.py:75-94`) |
| 4.27 | GET /api/payments/{id}/sync-wise — stub | 0.5 | 0.25 | Stub returns placeholder (`payments.py:97-100`) |

**Payments Subtotal: 2.0/6**

### Inventory (12 points)
| # | Criterion | Points | Score | Evidence |
|---|-----------|--------|-------|---------|
| 4.28 | GET /api/inventory/ — list with status | 1.5 | 0 | Returns `501 Not Implemented` (`inventory.py:33`) |
| 4.29 | GET /api/inventory/?status_filter=LOW | 1 | 0 | Returns 501 |
| 4.30 | GET /api/inventory/stats — 5 aggregates | 1.5 | 0 | Returns 501 |
| 4.31 | GET /api/inventory/{product_id} — auto-create | 1 | 0 | Returns 501 |
| 4.32 | GET /api/inventory/{product_id}/movements | 0.5 | 0 | Returns 501 |
| 4.33 | POST /api/inventory/receive-shipment | 1.5 | 0 | Returns 501 |
| 4.34 | POST /api/inventory/sale | 1.5 | 0 | Returns 501 |
| 4.35 | POST /api/inventory/bulk-sale | 1.5 | 0 | Returns 501 |
| 4.36 | POST /api/inventory/adjust | 1.5 | 0 | Returns 501 |

**Inventory Subtotal: 0/12**

### Dashboard (4 points)
| # | Criterion | Points | Score | Evidence |
|---|-----------|--------|-------|---------|
| 4.37 | GET /api/dashboard/ — 4 KPIs | 2 | 0 | Returns `501 Not Implemented` (`dashboard.py:8`) |
| 4.38 | GET /api/dashboard/order-pipeline | 1 | 0 | Returns 501 (`dashboard.py:13`) |
| 4.39 | GET /api/dashboard/inventory-health | 1 | 0 | Returns 501 (`dashboard.py:18`) |

**Dashboard Subtotal: 0/4**

**Section Total: 9/50**

---

### Section 5: Business Logic Correctness (25 points)

| # | Criterion | Max | Score | Evidence |
|---|-----------|-----|-------|---------|
| 5.1 | PO status workflow enforces forward-only | 4 | 1 | `_validate_status_transition` checks `new_idx == current_idx + 1` (`orders.py:27-37`). Code correct but untestable (broken import). |
| 5.2 | All 10 PO stages work in order | 3 | 0.5 | All 10 stages defined in `PO_STATUS_ORDER` (`orders.py:13-24`). Untestable. |
| 5.3 | Insufficient stock sale returns 400 | 3 | 0 | Inventory router returns 501 for all operations |
| 5.4 | Inventory status computed correctly | 3 | 0 | Not implemented |
| 5.5 | Atomic bulk operations | 4 | 0 | Not implemented (inventory returns 501) |
| 5.6 | Adjustment uses absolute quantity | 2 | 0 | Not implemented (returns 501). Pydantic schema has `new_quantity: int` field in `schemas/inventory.py:49` suggesting intent. |
| 5.7 | Days until stockout formula | 2 | 0.5 | Function exists but uses `sales_velocity=1.0` default instead of querying 90-day sales data (`inventory_monitor.py:13-14`). Wrong formula. |
| 5.8 | Alert deduplication — 24h | 2 | 1 | In-memory `_last_alert_sent` dict with 24h window check (`inventory_monitor.py:21-25`) |
| 5.9 | PO totals auto-calculated from line items | 2 | 0 | `create_order` creates line items but **does NOT calculate subtotal** (`orders.py:75-105`) |

**Section Total: 3/25**

---

### Section 6: Security (20 points)

| # | Criterion | Max | Score | Evidence |
|---|-----------|-----|-------|---------|
| 6.1 | Passwords hashed with bcrypt | 3 | 2 | `CryptContext(schemes=["bcrypt"], deprecated="auto")` used in `auth.py:13`. Hash would start with `$2b$`. |
| 6.2 | Bcrypt work factor 12 | 1 | 0 | No explicit `rounds=12` parameter. Default `CryptContext` bcrypt rounds are 12 in passlib, but not explicitly set. Giving 0 for ambiguity. |
| 6.3 | JWT access tokens expire in 15 minutes | 1.5 | 1.5 | `ACCESS_TOKEN_EXPIRE_MINUTES: int = 15` in `config.py:9`. Used in `create_access_token` (`auth.py:41-42`). |
| 6.4 | JWT uses HS256 | 0.5 | 0.5 | `ALGORITHM: str = "HS256"` in `config.py:8`. Used in `jwt.encode(..., algorithm=settings.ALGORITHM)` (`auth.py:45-47`). |
| 6.5 | Refresh tokens with 7-day expiry | 1 | 1 | `REFRESH_TOKEN_EXPIRE_DAYS: int = 7` in `config.py:10`. Used in `create_refresh_token` (`auth.py:53`). |
| 6.6 | TOTP MFA with pyotp | 2 | 2 | `import pyotp` (`auth.py:6`). `pyotp.random_base32()` for secret, `TOTP(secret).provisioning_uri()` for QR, `TOTP(secret).verify(code)` for validation (`auth.py:113,116-118,127-128`). |
| 6.7 | Rate limiting: auth returns 429 after 5 rapid attempts | 3 | 0 | No auth-specific rate limit (`@limiter.limit("5/minute")` or similar not applied to auth endpoints). Backend can't start anyway. |
| 6.8 | Global rate limiting: 100/min | 1 | 0.5 | `Limiter(key_func=get_remote_address, default_limits=["100/minute"])` configured (`main.py:15`). Backend can't start to verify. |
| 6.9 | Security headers (all 4) | 2 | 1 | 3 of 4 correct headers in `SecurityHeadersMiddleware` (`main.py:41-44`): `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: no-referrer`. Has `Permissions-Policy` instead of `Content-Security-Policy`. → 3/4 × 2 = 1.5, rounded to 1. |
| 6.10 | CORS not set to wildcard | 1 | 1 | Specific origins list: `["https://inventory.yourdomain.com", "http://localhost:3000"]` (`main.py:24-27`) |
| 6.11 | Fernet encryption for sensitive fields | 2 | 0 | `DB_ENCRYPTION_KEY: str` in config (`config.py:7`) but Fernet is never imported or used anywhere in the codebase |
| 6.12 | Bleach for input sanitization | 1 | 0 | No `bleach` import or usage anywhere. Not in `pyproject.toml` dependencies. |
| 6.13 | No raw SQL with user input | 1 | 1 | All database access uses SQLAlchemy ORM. No `text()` calls found in any router. |

**Section Total: 10.5/20**

---

### Section 7: Audit Trail (10 points)

| # | Criterion | Max | Score | Evidence |
|---|-----------|-----|-------|---------|
| 7.1 | audit_log records after CREATE | 2 | 0.5 | `log_action` handles creation case — logs non-sensitive fields with `{old: None, new: val}` format (`services/audit.py:34-39`). But never actually called during product/supplier/order creation. Only integration is in `payments.py:91`. |
| 7.2 | audit_log after UPDATE with old+new | 2 | 1 | `_filter_changes` compares old record attributes with new values (`audit.py:11-19`). Called from `payments.py:91` for payment updates. |
| 7.3 | audit_log after DELETE | 1 | 0 | No DELETE case in `log_action`. Products soft-delete doesn't call audit. |
| 7.4 | password_hash and mfa_secret excluded | 2 | 1.5 | `SENSITIVE_FIELDS = {"password_hash", "mfa_secret"}` explicitly filtered in both `_filter_changes` and creation logging (`audit.py:8,14,38`) |
| 7.5 | No-op updates don't create entries | 1.5 | 1 | `if not changes: return` — early exit when nothing changed (`audit.py:40-41`) |
| 7.6 | IP address and user-agent captured | 1.5 | 0 | `log_action` signature has no `request` parameter. `AuditLog` model has no `ip_address` or `user_agent` columns. Not captured. |

**Section Total: 4/10**

---

### Section 8: Frontend (10 points)

| # | Criterion | Max | Score | Evidence |
|---|-----------|-----|-------|---------|
| 8.1 | Page renders 4 KPI cards | 3 | 0 | JSX syntax error on `page.tsx:59` — `className={...`)}>` has extra `)`. Component **cannot compile**. 4 cards defined in `cards` array (`page.tsx:40-45`). |
| 8.2 | Cards show real data from API | 2 | 0 | Cannot render. `useEffect` fetch from `/api/dashboard/` exists (`page.tsx:24-38`). |
| 8.3 | Responsive grid (1->2->4) | 1.5 | 0.5 | Code has `grid-cols-1 sm:grid-cols-2 lg:grid-cols-4` (`page.tsx:57`). Correct breakpoints but won't render. |
| 8.4 | Loading state visible | 1 | 0.5 | SVG spinner with `animate-spin` in loading branch (`page.tsx:49-55`). Won't render. |
| 8.5 | Error state doesn't crash | 1 | 0.5 | `.catch(() => { setError(true); setLoading(false); })` with `defaultData` fallback to zeroes (`page.tsx:12-17,34-36`). Won't render. |
| 8.6 | TailwindCSS used | 0.5 | 0.5 | Tailwind classes throughout. `globals.css` has `@tailwind` directives. Dynamic classes like `bg-${card.color}-100` won't be purged by Tailwind though. |
| 8.7 | Light theme, correct color palette | 0.5 | 0.5 | `tailwind.config.ts` defines `primary: '#2563EB'` (blue), `success: '#10B981'` (green), `warning: '#F59E0B'` (amber), `danger: '#EF4444'` (red) |
| 8.8 | `layout.tsx` + `globals.css` exist | 0.5 | 0.25 | `globals.css` exists with Tailwind directives. **`layout.tsx` is MISSING** — required by Next.js App Router. |

**Section Total: 2.75/10**

---

## Final Score Card

| Section | Max | Score |
|---------|-----|-------|
| Section 1: Does It Run? | 30 | 6 |
| Section 2: Test Suite | 35 | 0 |
| Section 3: Schema Completeness | 20 | 1.5 |
| Section 4: API Endpoints | 50 | 9 |
| Section 5: Business Logic | 25 | 3 |
| Section 6: Security | 20 | 10.5 |
| Section 7: Audit Trail | 10 | 4 |
| Section 8: Frontend | 10 | 2.75 |
| **Total** | **200** | **36.75** |

**Rounded Total: 37/200**

**Final Grade: F (Fundamentally incomplete)**

---

## Score Distribution by Priority

| Priority | Sections | Max | Score | % Achieved |
|----------|----------|-----|-------|------------|
| Does it even work? | Sec 1 + Sec 2 | 65 | 6 | 9.2% |
| Core functionality | Sec 4 + Sec 5 | 75 | 12 | 16.0% |
| Data integrity | Sec 3 + Sec 7 | 30 | 5.5 | 18.3% |
| Security | Sec 6 | 20 | 10.5 | 52.5% |
| Frontend | Sec 8 | 10 | 2.75 | 27.5% |

---

## Key Observations

### Strongest Area: Security Configuration (52.5%)
Despite the application not running, the security patterns are well-implemented in code: bcrypt hashing, JWT with correct expiry, TOTP MFA via pyotp, CORS whitelist, security headers, ORM-only database access.

### Weakest Area: Runnability (9.2%)
The most fundamental requirement — the app must start — is not met. Import errors cascade through the module system, preventing any endpoint from being reached.

### Architecture vs Implementation Gap
The project shows strong architectural awareness (correct directory structure, separation of concerns, proper schema design for orders/payments) but lacks the critical connecting tissue: no database module, no models for 18+ tables, no test infrastructure. It reads as a well-planned project that ran out of implementation time.

### Missing Critical Components
1. `app/db.py` or `app/database.py` — the SQLAlchemy engine, session, and `get_db` dependency
2. `app/models/__init__.py` and `app/models/base.py` — the declarative base and model registry
3. 18+ SQLAlchemy model classes (users, products, suppliers, orders, payments, inventory, etc.)
4. `init_db.py` — seed data script
5. `conftest.py` — test configuration and fixtures
6. All 8 test files (117+ tests)
7. `alembic.ini` and migration scripts
8. `frontend/src/app/layout.tsx` — Next.js App Router layout
9. Inventory router implementation (8 endpoints)
10. Dashboard router implementation (3 endpoints)
