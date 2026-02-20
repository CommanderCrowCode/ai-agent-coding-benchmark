# Agent 3: Backend - Business Logic

## Project Context
You are building a Small Business Inventory Management System. Tech stack: FastAPI (Python 3.11+), SQLAlchemy, PostgreSQL.

## IMPORTANT: Fully Autonomous Operation
The user is NOT available until the project is complete to spec. You MUST:
- Make ALL decisions yourself or by coordinating with teammates via relay-mesh
- NEVER wait for user input — if you have questions, ask the team lead or relevant teammate via `send_message`
- Keep working until your deliverables are done and the team lead confirms completion
- Check `fetch_messages` regularly (after each deliverable, every few minutes while waiting)

## Your Responsibility
Build orders, payments, inventory, and dashboard routers plus background services.

## Deliverables

### 1. Orders Router (`webapp/backend/app/routers/orders.py`)
- GET /api/orders - List (filter by status, supplier_id)
- GET /api/orders/{id} - Detail with line_items and status_history
- GET /api/orders/workflow-stats - Count per status
- POST /api/orders - Create with line items; auto-generate PO-YYYY-NNN
- PUT /api/orders/{id} - Update notes, expected_ready_date
- POST /api/orders/{id}/status - Advance status with validation

**PO Status Workflow (10 stages):**
DRAFT → SENT → CONFIRMED → IN_PRODUCTION → SHIPPED → IN_TRANSIT → CUSTOMS → RECEIVED → QCD → STOCKED

Validate forward-only transitions.

### 2. Payments Router (`webapp/backend/app/routers/payments.py`)
- GET /api/payments - List (filter by status)
- GET /api/payments/{id} - Detail
- GET /api/payments/summary - Monthly totals, YTD totals, total fees
- POST /api/payments - Create; auto-generate PAY-YYYY-NNN
- PUT /api/payments/{id} - Update status, notes; log to audit_log
- GET /api/payments/{id}/sync-wise - Stub for Wise API sync

### 3. Inventory Router (`webapp/backend/app/routers/inventory.py`)
- GET /api/inventory/ - List all with stock status (OK/LOW/OUT_OF_STOCK)
- GET /api/inventory/stats - Aggregate stats
- GET /api/inventory/{product_id} - Detail; auto-create if missing
- GET /api/inventory/{product_id}/movements - Movement history
- POST /api/inventory/receive-shipment - Bulk receive (atomic)
- POST /api/inventory/sale - Record single sale (validate stock)
- POST /api/inventory/bulk-sale - Multiple sales atomically
- POST /api/inventory/adjust - Manual adjustment (absolute quantity)

### 4. Dashboard Router (`webapp/backend/app/routers/dashboard.py`)
- GET /api/dashboard/ - KPIs: total_inventory_value, pending_orders, low_stock_alerts, mtd_revenue
- GET /api/dashboard/order-pipeline - Orders grouped by status
- GET /api/dashboard/inventory-health - Per-SKU status summary

### 5. Services

#### Audit Service (`webapp/backend/app/services/audit.py`)
- log_action() function to log CREATE/UPDATE/DELETE
- Exclude sensitive fields: password_hash, mfa_secret
- Only log when values actually change

#### Inventory Monitor (`webapp/backend/app/services/inventory_monitor.py`)
- _calculate_days_until_stockout() - Use 90-day sales velocity
- _should_send_alert() - Check 24-hour deduplication
- check_low_stock_and_alert() - Send email, log to notifications

### 6. Email Utility (`webapp/backend/app/utils/email.py`)
- send_low_stock_alert() - HTML email with styled alert card

### 7. Background Tasks (`webapp/backend/app/tasks/scheduler.py`)
- Run inventory_monitor.check_low_stock_and_alert() every hour via asyncio

## relay-mesh Registration

When registering with relay-mesh, use these **exact values**:
- `name`: "backend-logic"
- `project`: "inventory-management"
- `role`: "backend-engineer"
- `specialization`: "business-logic"
- `description`: "Backend engineer building orders, payments, inventory, and dashboard APIs"

### After registration completes
1. Call `list_agents` to discover all registered teammates
2. Call `send_message` to introduce yourself to the team lead (look for role "team-lead"). If no lead found yet, call `broadcast_message` instead
3. Call `fetch_messages` to check if anyone has already sent you work or instructions

### Continuous Coordination
- After completing each deliverable, call `fetch_messages` and report progress to the team lead via `send_message`
- If you have questions or blockers, ask the team lead or relevant teammate — do NOT wait for user input
- When all your deliverables are done, send a completion message to the team lead listing what you built
- Keep checking `fetch_messages` even after your work is done — the team lead may ask for fixes or changes
- Coordinate with the team lead for models/schemas
- Coordinate with backend-auth agent for main.py integration
- Report completion status via `broadcast_message`

## Success Criteria
- All 10 PO status transitions work correctly
- Inventory bulk operations are atomic (rollback on failure)
- Sale beyond available stock returns 400
- Low stock products appear in filter
- All mutations logged to audit_log
