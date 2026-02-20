# Prompt: Build a Small Business Inventory Management System

Build a full-stack inventory management web application for a small import/wholesale business that sources products from overseas suppliers (primarily via Alibaba/direct factory) and sells through e-commerce platforms (Shopee, Lazada) and direct channels.

---

## Business Context

The business imports developmental toy kits for babies/toddlers (0–24 months), sold in 8 SKU variants covering different age ranges. Products are sourced from a single primary Chinese manufacturer with 45-day lead times. Payments are made via international wire transfer (30% deposit, 70% before shipment). Goods arrive by sea freight to Thailand.

---

## Tech Stack

- **Backend:** FastAPI (Python 3.11+) with SQLAlchemy ORM
- **Database:** PostgreSQL (production), SQLite (development/testing)
- **Migrations:** Alembic
- **Frontend:** Next.js 14 with React 18 and TailwindCSS
- **Auth:** JWT (HS256, 15-min expiry) + TOTP-based MFA
- **Infrastructure:** Docker Compose with Nginx reverse proxy
- **Testing:** pytest (aim for 100+ tests, all passing)
- **Python tooling:** uv for all package management

---

## Database Schema

Design 20+ tables with the following structure:

### Auth & Security
- **users** — email, password_hash (bcrypt), mfa_secret, mfa_enabled, role, is_active
- **audit_log** — table_name, record_id, action (CREATE/UPDATE/DELETE), old_values (JSON), new_values (JSON), user_id, ip_address, user_agent, created_at
  - Exclude sensitive fields: password_hash, mfa_secret
  - Only log when values actually change

### Products & Suppliers
- **suppliers** — name, legal_name, website, address, payment_terms, lead_time_days, moq, notes
- **supplier_contacts** — supplier_id, name, role, platform (Alibaba, WeChat, etc.), contact_value
- **products** — internal_sku (LUMI-xxxx, unique), supplier_sku (BCTxxxx), name, age_range_start, age_range_end, price_usd, units_per_carton, is_best_seller, is_active, supplier_id
- **price_history** — product_id, supplier_id, price_usd, effective_date

### Purchase Orders
- **purchase_orders** — po_number (PO-YYYY-NNN, unique), supplier_id, supplier_pi, order_date, status (enum), subtotal_usd, shipping_usd, other_fees_usd, total_usd, notes
- **po_line_items** — po_id, product_id, quantity, unit_price_usd
- **po_status_history** — po_id, old_status, new_status, changed_by, changed_at, notes

### Shipments
- **shipments** — po_id, carrier, tracking_number, bill_of_lading, origin_port, destination_port, status
- **shipment_milestones** — shipment_id, milestone_type (ORDER_PLACED/SHIPPED/ARRIVED/CLEARED/DELIVERED), expected_date, actual_date

### Payments
- **payments** — payment_id (PAY-YYYY-NNN, unique), po_id, supplier_id, initiated_at, wise_transfer_id, wise_tracking_id, status (PENDING/PROCESSING/DELIVERED/FAILED), amount_usd, fee_usd, amount_received, currency_received, notes

### Inventory
- **inventory** — product_id (unique FK), quantity, reorder_point (default 10), safety_stock_days (default 14), last_counted_at
- **stock_movements** — product_id, movement_type (IN/OUT/ADJUSTMENT), quantity, reference_type (SHIPMENT/SALE/ADJUSTMENT), reference_id, notes, created_at
- **sales** — product_id, sale_date, quantity, unit_price_usd, platform (SHOPEE/LAZADA/DIRECT), order_reference

### Planning
- **seasonality_factors** — month (1–12), multiplier (e.g., 1.30 for December)
- **holidays** — country, holiday_name, start_date, end_date, type (FACTORY_CLOSURE/HIGH_DEMAND)
- **settings** — key, value (system config key-value store)
- **notifications** — product_id, alert_type, message, status (PENDING/SENT), created_at

### Key Indexes
```sql
idx_products_internal_sku (unique)
idx_purchase_orders_status
idx_purchase_orders_order_date
idx_payments_status
idx_payments_wise_transfer_id
idx_stock_movements_product_date (composite: product_id, created_at)
idx_sales_product_date (composite: product_id, sale_date)
idx_price_history_product_date
idx_notifications_status
idx_audit_log_table_record (composite: table_name, record_id)
idx_audit_log_created_at
```

---

## Backend API (FastAPI)

### Application Setup (`app/main.py`)
- FastAPI with lifespan manager for background tasks
- CORS: allow `https://inventory.yourdomain.com` and `http://localhost:3000`
- Global rate limiter: 100 req/min
- Auth endpoint rate limit: 5 req/min
- Security headers middleware:
  - `X-Content-Type-Options: nosniff`
  - `X-Frame-Options: DENY`
  - `Referrer-Policy: strict-origin-when-cross-origin`
  - `Content-Security-Policy: default-src 'self'`
- Health check: `GET /health` → `{"status": "ok"}`

### Configuration (`app/config.py`)
Pydantic Settings class reading from env vars:
- `DATABASE_URL`, `SECRET_KEY`, `DB_ENCRYPTION_KEY`, `ALGORITHM`
- `ACCESS_TOKEN_EXPIRE_MINUTES=15`, `REFRESH_TOKEN_EXPIRE_DAYS=7`
- `WISE_API_KEY`, `WISE_WEBHOOK_SECRET`
- `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASSWORD`, `FROM_EMAIL`
- `DEBUG`

### Route Modules

#### `routers/auth.py`
- `POST /api/auth/register` — Create user, hash password with bcrypt
- `POST /api/auth/login` — Verify credentials, return JWT access + refresh tokens
- `POST /api/auth/refresh` — Exchange refresh token for new access token
- `POST /api/auth/mfa/setup` — Generate TOTP secret, return QR code URI
- `POST /api/auth/mfa/verify` — Validate TOTP code, enable MFA on account

#### `routers/products.py`
- `GET /api/products` — List (filter by `best_seller`, `active`)
- `GET /api/products/{id}` — Detail with supplier info
- `POST /api/products` — Create, record price in price_history
- `PUT /api/products/{id}` — Update; if price changed, record in price_history
- `DELETE /api/products/{id}` — Soft delete (`is_active = False`)

#### `routers/suppliers.py`
- `GET /api/suppliers` — List all
- `GET /api/suppliers/{id}` — Detail with contacts
- `GET /api/suppliers/{id}/contacts` — List contacts only
- `POST /api/suppliers` — Create
- `PUT /api/suppliers/{id}` — Update

#### `routers/orders.py`
- `GET /api/orders` — List (filter by `status`, `supplier_id`)
- `GET /api/orders/{id}` — Detail with line_items and status_history
- `GET /api/orders/workflow-stats` — Count per status
- `POST /api/orders` — Create with line items; auto-generate `PO-YYYY-NNN`; calculate totals from line items
- `PUT /api/orders/{id}` — Update notes, expected_ready_date
- `POST /api/orders/{id}/status` — Advance status with validation; record in po_status_history

**PO Status Workflow (10 stages in order):**
```
DRAFT → SENT → CONFIRMED → IN_PRODUCTION → SHIPPED → IN_TRANSIT → CUSTOMS → RECEIVED → QCD → STOCKED
```
Validate that status transitions are forward-only (no skipping back to DRAFT from STOCKED).

#### `routers/payments.py`
- `GET /api/payments` — List (filter by `status`)
- `GET /api/payments/{id}` — Detail
- `GET /api/payments/summary` — Monthly totals, YTD totals, total fees
- `POST /api/payments` — Create; auto-generate `PAY-YYYY-NNN`
- `PUT /api/payments/{id}` — Update status, notes; log to audit_log
- `GET /api/payments/{id}/sync-wise` — Stub for Wise API sync (return current data)

#### `routers/inventory.py` (most complex)
- `GET /api/inventory/` — List all products with stock status
  - Status: `OK` (qty > reorder_point), `LOW` (0 < qty ≤ reorder_point), `OUT_OF_STOCK` (qty == 0)
  - Filter: `?status_filter=LOW` or `?status_filter=OUT_OF_STOCK`
  - Use `joinedload` for product relationship
- `GET /api/inventory/stats` — Aggregate: total_value, total_products, in_stock_count, low_stock_count, out_of_stock_count
- `GET /api/inventory/{product_id}` — Detail; auto-create inventory record if missing (quantity=0)
- `GET /api/inventory/{product_id}/movements` — Movement history with pagination
- `POST /api/inventory/receive-shipment` — Bulk receive
  - Body: `{"items": [{"product_id": 1, "quantity": 20}], "shipment_reference": "SHIP-001"}`
  - Validate all products exist BEFORE processing (atomic)
  - Create `IN` stock movement for each item
  - Update inventory quantity and `updated_at`
- `POST /api/inventory/sale` — Record single sale
  - Validate sufficient stock
  - Create `OUT` movement
  - Create sale record (with platform, order_reference)
  - Decrement inventory
- `POST /api/inventory/bulk-sale` — Multiple sales atomically
  - Pre-validate ALL items for stock before processing any
  - If any fail, roll back all
- `POST /api/inventory/adjust` — Manual adjustment
  - Body: `{"product_id": 1, "quantity": 45, "reason": "physical count"}`
  - Quantity is absolute (target stock level), not delta
  - Prevent resulting in negative inventory
  - Create `ADJUSTMENT` movement with delta
  - Update `last_counted_at`

#### `routers/dashboard.py`
- `GET /api/dashboard/` — KPIs: total_inventory_value, pending_orders, low_stock_alerts, mtd_revenue
- `GET /api/dashboard/order-pipeline` — Orders grouped by status
- `GET /api/dashboard/inventory-health` — Per-SKU status summary

### Services

#### `services/audit.py`
```python
def log_action(db, user_id, table_name, record_id, action, old_values, new_values, request):
    # Exclude sensitive fields from logging
    EXCLUDED_FIELDS = {"password_hash", "mfa_secret"}
    # Detect actual changes: only log if old != new
    # Capture request.client.host for IP, request.headers.get("user-agent")
```

#### `services/inventory_monitor.py`
```python
class InventoryMonitor:
    def _calculate_days_until_stockout(self, db, product_id, current_stock) -> float:
        # Query sales in last 90 days
        # Calculate daily_velocity = total_sold / 90
        # Return current_stock / daily_velocity (or infinity if no sales)

    def _should_send_alert(self, db, product_sku) -> bool:
        # Check notifications table: was alert sent in last 24 hours?

    def check_low_stock_and_alert(self, db):
        # For each LOW/OUT_OF_STOCK product
        # If _should_send_alert: send email, log to notifications
```

#### `utils/email.py`
```python
def send_low_stock_alert(product_sku, current_qty, reorder_point, days_until_stockout, recipients):
    # HTML email with styled alert card
    # Include: SKU, current qty, reorder point, days until stockout
    # Action buttons: "Create Purchase Order", "View Inventory"
    # Send via SMTP
```

### Background Tasks (`tasks/scheduler.py`)
- Run `inventory_monitor.check_low_stock_and_alert()` every hour via asyncio lifespan

---

## Frontend (Next.js 14)

### Dashboard Page (`src/app/page.tsx`)
- Client component (`'use client'`)
- Fetches `GET /api/dashboard/kpis` on mount
- 4 KPI cards in responsive grid (1→2→4 columns):
  - Total Inventory Value (USD)
  - Pending Orders
  - Low Stock Alerts
  - Month-to-Date Revenue
- 2 sections below: Order Pipeline, Inventory Health (placeholder "Coming soon...")
- Loading spinner state, error fallback with zeroes

### Styling
- TailwindCSS throughout
- Light theme only (white background, dark text)
- Color palette: Blue primary, Green success, Amber warning, Red danger

---

## Infrastructure (Docker Compose)

Three services:
1. **frontend** — Next.js, port 3000, 0.5 CPU / 512MB RAM
2. **backend** — FastAPI/uvicorn, port 8000, 0.75 CPU / 1GB RAM, health check every 30s
3. **nginx** — Reverse proxy, ports 80 + 443, SSL termination

Custom bridge network. Backend reads from `.env` file.

### Nginx Config
- Proxy `/api/` to backend:8000
- Proxy `/` to frontend:3000
- SSL with certs in `./certs/`

---

## Testing (pytest)

Write 117+ tests covering:

| File | Tests | What to Cover |
|------|-------|---------------|
| test_products.py | 10 | CRUD, soft delete, price history |
| test_orders.py | 9 | Create, status transitions, line item totals |
| test_payments.py | 11 | CRUD, summary endpoint, status update |
| test_inventory.py | 22 | List/filter, receive shipment, sale, bulk sale, adjust, movements, status determination |
| test_suppliers.py | 10 | CRUD, contacts |
| test_auth.py | 19 | Login, MFA setup/verify, refresh, rate limiting, security headers |
| test_audit.py | 18 | CREATE/UPDATE/DELETE logging, old/new values, sensitive field exclusion, change detection |
| test_inventory_monitor.py | 18 | Days-until-stockout calc, alert deduplication (24h), email trigger |

Test infrastructure:
- In-memory SQLite for test isolation
- Autouse fixtures for setup/teardown
- Override `get_db()` dependency per test
- pytest-asyncio for async tests

---

## Security Requirements

- Passwords: bcrypt with work factor 12
- JWT: HS256, 15-minute access tokens, 7-day refresh tokens
- MFA: TOTP (pyotp), QR code setup flow
- Rate limiting: 100/min global, 5/min on auth endpoints (slowapi)
- Input sanitization: bleach for any HTML input
- Encrypted storage: Fernet encryption for sensitive fields (bank account numbers)
- No SQL injection: use ORM only, no raw queries with user input
- CORS: whitelist only

---

## Database Initialization Script

`webapp/backend/init_db.py` should seed:
1. Admin user: `admin@example.com` / `admin123`
2. One supplier with contacts
3. 4–8 sample products
4. 2 sample purchase orders (one CONFIRMED, one DRAFT) with line items
5. Inventory records for 2 products (one healthy, one low stock)
6. Sample stock movements (IN and OUT) for audit trail

---

## Key Business Logic

### Inventory Status Rules
- `OUT_OF_STOCK`: quantity == 0
- `LOW`: 0 < quantity ≤ reorder_point
- `OK`: quantity > reorder_point

### Days Until Stockout Formula
```
daily_velocity = total_sold_last_90_days / 90
days_until_stockout = current_quantity / daily_velocity
```

### Alert Deduplication
Only send one email alert per product per 24-hour window. Check `notifications` table before sending.

### PO Number Generation
Format: `PO-YYYY-NNN` (e.g., `PO-2026-001`). Query max existing number for the year, increment by 1.

### Payment ID Generation
Format: `PAY-YYYY-NNN` (e.g., `PAY-2026-001`). Same pattern as PO numbers.

### Atomic Bulk Operations
For `receive-shipment` and `bulk-sale`: validate ALL items before modifying ANY. Use database transactions. Rollback on any failure.

---

## Project Structure

```
inventory_management/
├── webapp/
│   ├── backend/
│   │   ├── pyproject.toml
│   │   ├── alembic.ini
│   │   ├── alembic/
│   │   ├── init_db.py
│   │   ├── app/
│   │   │   ├── main.py
│   │   │   ├── config.py
│   │   │   ├── database.py
│   │   │   ├── models.py
│   │   │   ├── routers/
│   │   │   │   ├── auth.py
│   │   │   │   ├── products.py
│   │   │   │   ├── orders.py
│   │   │   │   ├── payments.py
│   │   │   │   ├── inventory.py
│   │   │   │   ├── dashboard.py
│   │   │   │   └── suppliers.py
│   │   │   ├── schemas/
│   │   │   │   ├── products.py
│   │   │   │   ├── orders.py
│   │   │   │   ├── payments.py
│   │   │   │   └── suppliers.py
│   │   │   ├── services/
│   │   │   │   ├── audit.py
│   │   │   │   └── inventory_monitor.py
│   │   │   ├── utils/
│   │   │   │   └── email.py
│   │   │   └── tasks/
│   │   │       └── scheduler.py
│   │   └── tests/
│   │       ├── test_products.py
│   │       ├── test_orders.py
│   │       ├── test_payments.py
│   │       ├── test_inventory.py
│   │       ├── test_suppliers.py
│   │       ├── test_auth.py
│   │       ├── test_audit.py
│   │       └── test_inventory_monitor.py
│   ├── frontend/
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   ├── tailwind.config.ts
│   │   ├── next.config.js
│   │   └── src/app/
│   │       ├── page.tsx
│   │       ├── layout.tsx
│   │       └── globals.css
│   └── infrastructure/
│       ├── docker-compose.yml
│       └── nginx/
│           └── nginx.conf
```

---

## Python Dependencies (pyproject.toml)

```toml
[project]
requires-python = ">=3.11"
dependencies = [
    "fastapi>=0.109.0",
    "uvicorn[standard]>=0.27.0",
    "sqlalchemy>=2.0.25",
    "psycopg2-binary>=2.9.9",
    "alembic>=1.13.1",
    "pydantic>=2.5.3",
    "pydantic-settings>=2.1.0",
    "python-jose[cryptography]>=3.3.0",
    "passlib[bcrypt]>=1.7.4",
    "httpx>=0.26.0",
    "cryptography>=41.0.7",
    "pyotp>=2.9.0",
    "slowapi>=0.1.9",
    "bleach>=6.1.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.4.4",
    "pytest-cov>=4.1.0",
    "pytest-asyncio>=0.23.3",
    "ruff>=0.1.11",
]
```

---

## Acceptance Criteria

1. All 117+ pytest tests pass
2. `GET /health` returns 200
3. Can create a supplier, product, purchase order, and advance it through all 10 stages
4. Can receive a shipment and see inventory update
5. Recording a sale beyond available stock returns a 400 error
6. Low stock products appear in `GET /api/inventory/?status_filter=LOW`
7. All mutations (create/update/delete) appear in the audit_log
8. Auth rate limiting returns 429 after 5 login attempts per minute
9. Docker Compose brings up all 3 services successfully
10. Frontend dashboard loads and displays KPI cards

