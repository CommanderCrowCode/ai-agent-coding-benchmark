# Absolute Grading Report: Emergent

## Summary
- **Total Score: 101/200**
- **Grade: D**
- **Interpretation:** Partial implementation, many rough edges. Core backend logic is strong but the project doesn't run, has zero tests, no infrastructure, and a non-functional frontend.

## Critical Issues

1. **Backend cannot start** — `server.py` imports from `routers.auth`, `routers.products`, etc. (`server.py:12-17`), but the `routers/` directory is **completely empty**. The actual router files (auth.py, products.py, etc.) are at the `backend/` root level. This is a fatal import error.
2. **MongoDB instead of SQLAlchemy/PostgreSQL** — The entire backend uses MongoDB via `motor` (`server.py:5,25-26`) instead of the specified SQLAlchemy ORM + PostgreSQL.
3. **React CRA instead of Next.js 14** — Frontend is Create React App with craco, not Next.js 14.
4. **Zero test files** — No tests directory, no test files, no conftest.py.
5. **No infrastructure files** — No Docker Compose, no Nginx config.

---

## Section Scores

### Section 1: Does It Run? (30 points)

| # | Criterion | Max | Score | Evidence |
|---|-----------|-----|-------|----------|
| 1.1 | Backend starts without crashing | 5 | 0 | `server.py:12` does `from routers.auth import router as auth_router` but `routers/` directory is empty (verified via `ls -la backend/routers/` — only `.` and `..`). Fatal `ModuleNotFoundError` on startup |
| 1.2 | GET /health returns 200 | 3 | 0 | Backend can't start. Also endpoint is at `/api/health` not `/health` (`server.py:88`) |
| 1.3 | Frontend starts without crashing | 5 | 3 | CRA-based app with `"start": "craco start"` (`package.json:60`). Source files appear valid. Uses jsconfig path aliases + craco. Should compile, but is React CRA not Next.js |
| 1.4 | Frontend page loads in browser | 3 | 2 | Would render "Building something incredible ~!" (`App.js:35`) — something visible but not the specified dashboard |
| 1.5 | Database tables get created | 5 | 0 | Requires backend startup which fails. MongoDB collections would be auto-created on seed, but lifespan can't run |
| 1.6 | init_db.py runs and seeds data | 4 | 0 | **No standalone `init_db.py` exists.** Seeding is embedded in server lifespan (`server.py:37`) via `seed_database()` (`services/seed.py`). Cannot run independently |
| 1.7 | Docker Compose valid YAML, 3 services | 3 | 0 | **No `docker-compose.yml` file exists anywhere in the project** |
| 1.8 | Nginx config parses | 2 | 0 | **No nginx config file exists** |

**Section Total: 5/30**

---

### Section 2: Test Suite (35 points)

| # | Criterion | Max | Score | Evidence |
|---|-----------|-----|-------|----------|
| 2.1 | Test suite runs at all | 3 | 0 | **Zero test files exist.** `find emergent -name "test_*"` returns nothing. No `tests/` directory with any `.py` files. No `conftest.py` |
| 2.2 | Number of tests collected | 10 | 0 | 0 tests → 0 points |
| 2.3 | Pass rate | 12 | 0 | N/A — no tests to run |
| 2.4 | All 8 test files exist | 4 | 0 | None of the 8 required test files exist: test_products, test_orders, test_payments, test_inventory, test_suppliers, test_auth, test_audit, test_inventory_monitor |
| 2.5 | Test isolation | 3 | 0 | N/A |
| 2.6 | In-memory SQLite with get_db override | 3 | 0 | N/A — also uses MongoDB not SQLAlchemy |

**Section Total: 0/35**

---

### Section 3: Schema Completeness (20 points)

**Note:** Project uses MongoDB collections (implicit schema) instead of SQLAlchemy models. Scoring based on document structures used throughout the codebase.

| # | Criterion | Max | Score | Evidence |
|---|-----------|-----|-------|----------|
| 3.1 | Table count | 5 | 1 | 14 collections used: users, audit_log, products, suppliers, supplier_contacts, price_history, purchase_orders, po_line_items, po_status_history, payments, inventory, stock_movements, sales, notifications. Missing: shipments, shipment_milestones, seasonality_factors, holidays, settings. 10-14 → 1 point |
| 3.2 | All enum types properly defined (5+) | 3 | 1 | No formal Python Enum types. Status values stored as plain strings. PO_STATUSES (`orders.py:18-21`) and PAYMENT_STATUSES (`payments.py:18`) are Python lists, not enums. Movement types (IN/OUT/ADJUSTMENT) and platforms (SHOPEE/LAZADA/DIRECT) are inline strings. ~4 logical groupings but no formal definitions |
| 3.3 | All FK relationships present | 3 | 1 | MongoDB has no FK enforcement. Manual lookups exist (e.g., `db.products.find_one({"id": item.product_id})`) but no referential integrity guarantees |
| 3.4 | Unique constraints on internal_sku, po_number, payment_id | 2 | 0.5 | Only `internal_sku` has unique index: `db.products.create_index("internal_sku", unique=True)` (`server.py:47`). po_number and payment_number rely on regex-based generation with no unique constraint |
| 3.5 | All 11 specified indexes | 3 | 3 | 13 indexes created in `server.py:47-59`: products.internal_sku(unique), products.is_active, purchase_orders.status, purchase_orders.order_date, payments.status, stock_movements.(product_id,created_at), sales.(product_id,sale_date), price_history.(product_id,effective_date), notifications.status, audit_log.(table_name,record_id), audit_log.created_at, inventory.product_id(unique), users.email(unique) |
| 3.6 | Default values (reorder_point=10, safety_stock_days=14) | 1 | 1 | Defaults used in inventory creation: `"reorder_point": 10, "safety_stock_days": 14` (`inventory.py:133-134`, `products.py:133-134`) |
| 3.7 | JSON columns for audit_log old/new values | 1 | 1 | MongoDB naturally stores dicts as JSON-like documents. `old_values` and `new_values` are Python dicts inserted directly (`services/audit.py:35-36`) |
| 3.8 | Proper timestamps (created_at, updated_at) | 2 | 2 | Present on products (`products.py:114-115`), suppliers (`suppliers.py:84`), orders (`orders.py:165-166`), payments (`payments.py:145-146`), inventory (`inventory.py:157`), audit_log (`services/audit.py:40`) |

**Section Total: 10.5/20**

---

### Section 4: API Endpoints — Do They Exist and Work? (50 points)

**Note:** All endpoint code exists and logic is correct, but the backend **cannot start** due to broken imports (`server.py` references `routers/` directory which is empty). Scoring based on code review since endpoints are non-functional as deployed.

### Auth (8 points)

| # | Criterion | Max | Score | Evidence |
|---|-----------|-----|-------|----------|
| 4.1 | POST /api/auth/register | 2 | 2 | Creates user with bcrypt rounds=12, returns access + refresh tokens (`auth.py:91-119`) |
| 4.2 | POST /api/auth/login | 2 | 2 | Verifies credentials, returns both JWT tokens with user info (`auth.py:122-139`) |
| 4.3 | POST /api/auth/refresh | 1.5 | 1.5 | Validates refresh token type, returns new access token (`auth.py:142-159`) |
| 4.4 | POST /api/auth/mfa/setup | 1.5 | 0 | **Not implemented.** No MFA endpoints. `pyotp` in requirements but never used |
| 4.5 | POST /api/auth/mfa/verify | 1 | 0 | **Not implemented** |

**Auth Total: 5.5/8**

### Products (6 points)

| # | Criterion | Max | Score | Evidence |
|---|-----------|-----|-------|----------|
| 4.6 | GET /api/products — filterable | 1.5 | 1.5 | `best_seller` and `active` query params (`products.py:44-57`) |
| 4.7 | GET /api/products/{id} with supplier | 1 | 1 | Fetches supplier if linked (`products.py:76-78`) |
| 4.8 | POST /api/products + price_history | 1.5 | 1.5 | Creates product and inserts price_history record (`products.py:89-127`) |
| 4.9 | PUT /api/products/{id} + price_history on change | 1.5 | 1.5 | Detects `price_usd` change, records in price_history (`products.py:160-167`) |
| 4.10 | DELETE soft delete | 0.5 | 0.5 | Sets `is_active: False` (`products.py:184`) |

**Products Total: 6/6**

### Suppliers (4 points)

| # | Criterion | Max | Score | Evidence |
|---|-----------|-----|-------|----------|
| 4.11 | GET /api/suppliers — list | 0.5 | 0.5 | Returns all sorted by name (`suppliers.py:47-51`) |
| 4.12 | GET /api/suppliers/{id} with contacts | 1 | 1 | Includes contacts and products (`suppliers.py:54-64`) |
| 4.13 | GET /api/suppliers/{id}/contacts | 0.5 | 0.5 | Dedicated contacts endpoint (`suppliers.py:67-71`) |
| 4.14 | POST /api/suppliers | 1 | 1 | With audit logging (`suppliers.py:74-89`) |
| 4.15 | PUT /api/suppliers/{id} | 1 | 1 | With change detection + audit (`suppliers.py:92-109`) |

**Suppliers Total: 4/4**

### Purchase Orders (10 points)

| # | Criterion | Max | Score | Evidence |
|---|-----------|-----|-------|----------|
| 4.16 | GET /api/orders — filterable | 1 | 1 | Both `status` and `supplier_id` filters (`orders.py:77-87`) |
| 4.17 | GET /api/orders/{id} with line_items + history | 1.5 | 1.5 | Both included with product enrichment (`orders.py:100-128`) |
| 4.18 | GET /api/orders/workflow-stats | 1 | 1 | MongoDB aggregation: count per status (`orders.py:63-73`) |
| 4.19 | POST /api/orders — creates with items, auto PO-YYYY-NNN, auto-calculates | 3 | 3 | Full implementation: validates supplier/products, generates PO number, calculates `subtotal = sum(qty * unit_price_usd)`, creates line items + status history (`orders.py:131-193`) |
| 4.20 | PUT /api/orders/{id} | 0.5 | 0.5 | Updates notes/fees, recalculates total (`orders.py:196-219`) |
| 4.21 | POST /api/orders/{id}/status | 3 | 3 | Forward-only enforcement: `new_idx <= current_idx` → 400. Records in po_status_history (`orders.py:222-260`) |

**Orders Total: 10/10**

### Payments (6 points)

| # | Criterion | Max | Score | Evidence |
|---|-----------|-----|-------|----------|
| 4.22 | GET /api/payments — filterable | 0.5 | 0.5 | Status filter works (`payments.py:75-85`) |
| 4.23 | GET /api/payments/{id} | 0.5 | 0.5 | Includes supplier and PO enrichment (`payments.py:98-112`) |
| 4.24 | GET /api/payments/summary | 2 | 1.5 | Has MTD, YTD, total_fees, pending_amount, payment_count (`payments.py:50-72`). No per-month breakdown |
| 4.25 | POST /api/payments — auto PAY-YYYY-NNN | 1.5 | 1.5 | Correct format with auto-increment (`payments.py:36-47`) |
| 4.26 | PUT /api/payments/{id} + audit | 1 | 1 | Updates with status validation + audit log (`payments.py:154-174`) |
| 4.27 | GET /api/payments/{id}/sync-wise stub | 0.5 | 0 | **Not implemented** — no sync-wise endpoint |

**Payments Total: 5.5/6**

### Inventory (12 points)

| # | Criterion | Max | Score | Evidence |
|---|-----------|-----|-------|----------|
| 4.28 | GET /api/inventory/ — computed status | 1.5 | 1.5 | `determine_status()` correctly returns OK/LOW/OUT_OF_STOCK (`inventory.py:46-51`) |
| 4.29 | GET /api/inventory/?status_filter=LOW | 1 | 1 | Filter works (`inventory.py:121-122`) |
| 4.30 | GET /api/inventory/stats — 5 fields | 1.5 | 1.5 | total_value, total_products, in_stock_count, low_stock_count, out_of_stock_count (`inventory.py:84-90`) |
| 4.31 | GET /api/inventory/{product_id} — auto-creates | 1 | 1 | Creates with qty=0 if not found (`inventory.py:149-160`) |
| 4.32 | GET /api/inventory/{product_id}/movements — paginated | 0.5 | 0.5 | skip/limit pagination with total count (`inventory.py:173-186`) |
| 4.33 | POST /api/inventory/receive-shipment | 1.5 | 1.5 | Bulk receive with pre-validation (`inventory.py:189-237`) |
| 4.34 | POST /api/inventory/sale | 1.5 | 1.5 | Stock validation + OUT movement + sale record (`inventory.py:240-287`) |
| 4.35 | POST /api/inventory/bulk-sale — atomic | 1.5 | 1.5 | Pre-validates all items before any modification (`inventory.py:290-345`) |
| 4.36 | POST /api/inventory/adjust — absolute target | 1.5 | 1.5 | Absolute quantity, `delta = data.quantity - current_qty` (`inventory.py:348-397`) |

**Inventory Total: 12/12**

### Dashboard (4 points)

| # | Criterion | Max | Score | Evidence |
|---|-----------|-----|-------|----------|
| 4.37 | GET /api/dashboard/ — 4 KPIs | 2 | 2 | total_inventory_value, pending_orders, low_stock_alerts, mtd_revenue (`dashboard.py:14-56`) |
| 4.38 | GET /api/dashboard/order-pipeline | 1 | 1 | Grouped by status with count + total_usd (`dashboard.py:59-78`) |
| 4.39 | GET /api/dashboard/inventory-health | 1 | 1 | Per-SKU summary: product info, qty, reorder_point, status, value (`dashboard.py:81-115`) |

**Dashboard Total: 4/4**

**Section Total: 47.5/50**

---

### Section 5: Business Logic Correctness (25 points)

| # | Criterion | Max | Score | Evidence |
|---|-----------|-----|-------|----------|
| 5.1 | PO status workflow forward-only | 4 | 4 | `current_idx = PO_STATUSES.index(current_status)`, `new_idx = PO_STATUSES.index(new_status)`, `if new_idx <= current_idx: raise HTTPException(400)` (`orders.py:235-239`) |
| 5.2 | All 10 PO stages work in order | 3 | 3 | All 10 defined: DRAFT→SENT→CONFIRMED→IN_PRODUCTION→SHIPPED→IN_TRANSIT→CUSTOMS→RECEIVED→QCD→STOCKED (`orders.py:18-21`) |
| 5.3 | Insufficient stock returns 400 | 3 | 3 | `HTTPException(status_code=400, detail=f"Insufficient stock...")` (`inventory.py:253`) |
| 5.4 | Inventory status computed correctly | 3 | 3 | `determine_status()`: qty==0→OUT_OF_STOCK, qty<=reorder→LOW, else OK (`inventory.py:46-51`) |
| 5.5 | Atomic bulk operations | 4 | 3 | Both receive-shipment and bulk-sale pre-validate all items before modifying. However, MongoDB operations are not wrapped in a transaction — a crash mid-processing would leave partial state. Pre-validation mitigates but doesn't guarantee true atomicity (`inventory.py:195-200`, `296-308`) |
| 5.6 | Adjustment uses absolute quantity | 2 | 2 | `delta = data.quantity - current_qty`, inventory set to `data.quantity` (`inventory.py:362`, `379`) |
| 5.7 | Days until stockout formula | 2 | 0 | **Not implemented** — no `inventory_monitor.py` exists |
| 5.8 | Alert deduplication — 24h | 2 | 0 | **Not implemented** |
| 5.9 | PO totals auto-calculated from line items | 2 | 2 | `subtotal = sum(item.quantity * item.unit_price_usd for item in data.line_items)` (`orders.py:150`) |

**Section Total: 20/25**

---

### Section 6: Security (20 points)

| # | Criterion | Max | Score | Evidence |
|---|-----------|-----|-------|----------|
| 6.1 | Passwords hashed with bcrypt | 3 | 3 | `bcrypt.hashpw(password.encode('utf-8'), bcrypt.gensalt(rounds=12))` (`auth.py:47`) |
| 6.2 | Bcrypt work factor 12 | 1 | 1 | `bcrypt.gensalt(rounds=12)` — explicit rounds=12 (`auth.py:47`) |
| 6.3 | JWT access tokens 15 min | 1.5 | 1.5 | `ACCESS_TOKEN_EXPIRE_MINUTES = 15` (`auth.py:14`) |
| 6.4 | JWT uses HS256 | 0.5 | 0.5 | `ALGORITHM = "HS256"` (`auth.py:13`) |
| 6.5 | Refresh tokens 7-day | 1 | 1 | `REFRESH_TOKEN_EXPIRE_DAYS = 7` (`auth.py:15`) |
| 6.6 | TOTP MFA with pyotp | 2 | 0 | `pyotp==2.9.0` in `requirements.txt:89` but **never imported or used in any source file**. No MFA setup/verify endpoints |
| 6.7 | Rate limiting: auth returns 429 | 3 | 0 | `slowapi==0.1.9` in `requirements.txt:109` but **never imported or configured**. No rate limiting anywhere |
| 6.8 | Global rate limiting 100/min | 1 | 0 | **Not implemented** |
| 6.9 | Security headers (all 4) | 2 | 1.5 | 3 of 4 present in middleware (`server.py:66-71`): X-Content-Type-Options: nosniff, X-Frame-Options: DENY, Referrer-Policy: strict-origin-when-cross-origin. **Missing: Content-Security-Policy** |
| 6.10 | CORS not wildcard | 1 | 0 | `CORS_ORIGINS="*"` in `backend/.env:3`. `allow_origins=os.environ.get('CORS_ORIGINS', '*').split(',')` (`server.py:76`) — resolves to `['*']` |
| 6.11 | Fernet encryption for sensitive fields | 2 | 0 | `cryptography==46.0.4` in requirements but **Fernet never imported or used**. Bank details stored in plaintext |
| 6.12 | Bleach for input sanitization | 1 | 0 | `bleach==6.3.0` in `requirements.txt:9` but **never imported or used in any source file** |
| 6.13 | No raw SQL with user input | 1 | 1 | Uses MongoDB ORM (motor) throughout. No `text()` or raw queries with user input |

**Section Total: 9.5/20**

---

### Section 7: Audit Trail (10 points)

| # | Criterion | Max | Score | Evidence |
|---|-----------|-----|-------|----------|
| 7.1 | audit_log records after CREATE | 2 | 2 | `log_action` called with action="CREATE" in: products (`products.py:139`), suppliers (`suppliers.py:87`), supplier_contacts (`suppliers.py:127`), orders (`orders.py:191`), payments (`payments.py:149`) |
| 7.2 | audit_log records after UPDATE with old+new | 2 | 2 | `log_action` called with both old_values and new_values in: products (`products.py:170`), suppliers (`suppliers.py:106`), orders (`orders.py:216`, `257`), payments (`payments.py:171`) |
| 7.3 | audit_log records after DELETE | 1 | 1 | Product soft delete calls `log_action` with action="DELETE" (`products.py:185`) |
| 7.4 | password_hash and mfa_secret excluded | 2 | 2 | `EXCLUDED_FIELDS = {"password_hash", "mfa_secret", "_id"}` (`services/audit.py:4`). Filtered from both old/new values (`services/audit.py:9-12`) |
| 7.5 | No-op updates don't create entries | 1.5 | 1.5 | Change detection: compares old vs new values, returns early if no changes: `if not changes: return` (`services/audit.py:15-22`) |
| 7.6 | IP address and user-agent captured | 1.5 | 0 | Code exists to capture from `request.client.host` and `request.headers.get("user-agent")` (`services/audit.py:26-28`). However, **no caller ever passes the `request` parameter** — all calls omit it: `await log_action(db, user_id, table, id, action, old, new)`. IP and user-agent are always None |

**Section Total: 8.5/10**

---

### Section 8: Frontend (10 points)

| # | Criterion | Max | Score | Evidence |
|---|-----------|-----|-------|----------|
| 8.1 | Page renders 4 KPI cards | 3 | 0 | **No KPI cards.** `App.js` renders a logo link and "Building something incredible ~!" text (`frontend/src/App.js:23-37`). No dashboard components |
| 8.2 | Cards show real data from API | 2 | 0 | **No cards exist.** Only API call is `GET /api/` (`App.js:12`) — not a dashboard endpoint |
| 8.3 | Responsive grid (1->2->4 columns) | 1.5 | 0 | **No grid layout.** Single centered column with `flex-direction: column` (`App.css:17`) |
| 8.4 | Loading state visible | 1 | 0 | **No loading indicator** |
| 8.5 | Error state doesn't crash | 1 | 0 | Basic `try/catch` on API call logs to console but shows no UI fallback (`App.js:14-15`) |
| 8.6 | TailwindCSS used | 0.5 | 0 | Tailwind configured (`tailwind.config.js`, `index.css` with @tailwind directives) and UI components exist (`src/components/ui/`) but App.js uses plain CSS classes from `App.css` |
| 8.7 | Light theme, correct color palette | 0.5 | 0 | **Dark theme.** `background-color: #0f0f10`, `color: white` (`App.css:14,20`). Despite `index.css` having light theme variables |
| 8.8 | layout.tsx and globals.css exist | 0.5 | 0 | No `layout.tsx` (CRA, not Next.js). Has `index.css` not `globals.css`. Has `jsconfig.json` not `tsconfig.json` |

**Section Total: 0/10**

---

## Final Score Card

| Section | Max | Score |
|---------|-----|-------|
| Section 1: Does It Run? | 30 | 5 |
| Section 2: Test Suite | 35 | 0 |
| Section 3: Schema Completeness | 20 | 10.5 |
| Section 4: API Endpoints | 50 | 47.5 |
| Section 5: Business Logic | 25 | 20 |
| Section 6: Security | 20 | 9.5 |
| Section 7: Audit Trail | 10 | 8.5 |
| Section 8: Frontend | 10 | 0 |
| **Total** | **200** | **101** |

**Final Score: 101/200 — Grade: D**

---

## Analysis

### Distribution of Points

| Category | Points Available | Points Earned | % |
|----------|-----------------|---------------|---|
| Does it work? (Sec 1+2) | 65 | 5 | 7.7% |
| Core functionality (Sec 4+5) | 75 | 67.5 | 90.0% |
| Data integrity (Sec 3+7) | 30 | 19 | 63.3% |
| Security (Sec 6) | 20 | 9.5 | 47.5% |
| Frontend (Sec 8) | 10 | 0 | 0% |

### Key Observation
The project shows a stark contrast between **code quality** and **deployment readiness**:
- **API logic is excellent** (47.5/50, 95%) — the business logic for all major endpoints is correctly implemented
- **Infrastructure is absent** (5/65 for Sections 1+2, 7.7%) — the code can't run, has no tests, no Docker, no Nginx
- **Frontend is a placeholder** (0/10) — no dashboard implementation at all

This suggests the agent focused exclusively on backend business logic while neglecting testing, deployment infrastructure, frontend implementation, and basic project structure. The backend code itself would likely work correctly if the files were in the right directory (a single structural fix), but as shipped, the project is non-functional.

### Security Libraries Installed But Never Used
The following appear in `requirements.txt` but are never imported or used in source code:
- `pyotp==2.9.0` — listed for TOTP MFA, never imported
- `slowapi==0.1.9` — listed for rate limiting, never imported
- `bleach==6.3.0` — listed for input sanitization, never imported
- `cryptography==46.0.4` (for Fernet) — listed but Fernet never used

This pattern suggests the dependencies were auto-generated or bulk-installed without actually implementing the features they support.
