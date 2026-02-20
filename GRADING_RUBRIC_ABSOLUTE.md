# Absolute Grading Rubric: Raw Development Output

## Philosophy

Grade the final product as-is. No excuses, no normalization, no "but they had less time." Missing feature = zero points. Broken feature = near-zero points. This rubric measures the raw development capability of the agent — what it actually shipped.

---

## Scoring: 200 Points Total

---

## Section 1: Does It Run? (30 points)

The most basic bar. Can you start the thing and hit it.

| # | Criterion | Points | How to Verify |
|---|-----------|--------|---------------|
| 1.1 | Backend starts without crashing | 5 | `uv run uvicorn app.main:app` — process stays alive for 10s |
| 1.2 | `GET /health` returns `{"status": "ok"}` with HTTP 200 | 3 | `curl localhost:8000/health` |
| 1.3 | Frontend starts without crashing | 5 | `npm run dev` — compiles and serves on :3000 |
| 1.4 | Frontend page loads in a browser (no white screen of death) | 3 | Open localhost:3000, page renders something meaningful |
| 1.5 | Database tables get created (migrations or auto-create) | 5 | At least 15 tables exist after startup |
| 1.6 | `init_db.py` runs and seeds data without error | 4 | `uv run python init_db.py` exits 0 |
| 1.7 | Docker Compose config is valid YAML and defines 3 services | 3 | `docker compose config --quiet` succeeds |
| 1.8 | Nginx config parses without error | 2 | `nginx -t -c nginx.conf` or manual inspection |

---

## Section 2: Test Suite (35 points)

Tests are the contract. If they pass, the code works. If they don't exist, you're guessing.

| # | Criterion | Points | How to Verify |
|---|-----------|--------|---------------|
| 2.1 | Test suite runs at all (`uv run pytest` doesn't immediately crash) | 3 | Exit code isn't a traceback before collection |
| 2.2 | Number of tests collected | 10 | 0-29→0, 30-59→2, 60-89→4, 90-116→7, 117+→10 |
| 2.3 | Pass rate | 12 | <50%→0, 50-69%→3, 70-84%→6, 85-94%→9, 95-100%→12 |
| 2.4 | All 8 test files exist | 4 | 0.5 per file: test_products, test_orders, test_payments, test_inventory, test_suppliers, test_auth, test_audit, test_inventory_monitor |
| 2.5 | Test isolation — no test depends on another test's state | 3 | Run `uv run pytest --randomly-seed=12345` (or reverse order) — same results |
| 2.6 | In-memory SQLite with proper `get_db` override | 3 | Inspect conftest.py or test fixtures |

---

## Section 3: Schema Completeness (20 points)

Count the tables. Count the columns. Count the constraints. No partial credit for "I was going to add that."

| # | Criterion | Points | How to Verify |
|---|-----------|--------|---------------|
| 3.1 | Table count | 5 | <10→0, 10-14→1, 15-19→3, 20+→5 |
| 3.2 | All enum types properly defined (PO status 10-stage, movement types, payment status, platform, milestone types) | 3 | Grep for enum definitions — need at least 5 distinct enums |
| 3.3 | All foreign key relationships present and correct | 3 | Inspect models — every FK in spec exists |
| 3.4 | Unique constraints on internal_sku, po_number, payment_id | 2 | Grep for `unique=True` or `UniqueConstraint` |
| 3.5 | All 11 specified indexes created | 3 | Grep for `Index(` — count matches |
| 3.6 | Default values (reorder_point=10, safety_stock_days=14) | 1 | Inspect inventory model |
| 3.7 | JSON columns for audit_log old_values/new_values | 1 | Inspect audit_log model |
| 3.8 | Proper timestamps (created_at, updated_at) on relevant tables | 2 | Inspect models for auto-timestamp columns |

---

## Section 4: API Endpoints — Do They Exist and Work? (50 points)

Hit every endpoint. Does it respond with the right shape? Does it do what it says?

### Auth (8 points)
| # | Criterion | Points |
|---|-----------|--------|
| 4.1 | POST /api/auth/register — creates user, returns success | 2 |
| 4.2 | POST /api/auth/login — returns access + refresh JWT tokens | 2 |
| 4.3 | POST /api/auth/refresh — new access token from refresh token | 1.5 |
| 4.4 | POST /api/auth/mfa/setup — returns TOTP secret + QR URI | 1.5 |
| 4.5 | POST /api/auth/mfa/verify — validates code, enables MFA | 1 |

### Products (6 points)
| # | Criterion | Points |
|---|-----------|--------|
| 4.6 | GET /api/products — lists, filterable by best_seller and active | 1.5 |
| 4.7 | GET /api/products/{id} — includes supplier info | 1 |
| 4.8 | POST /api/products — creates + writes price_history | 1.5 |
| 4.9 | PUT /api/products/{id} — updates + price_history on price change | 1.5 |
| 4.10 | DELETE /api/products/{id} — soft delete (is_active=False) | 0.5 |

### Suppliers (4 points)
| # | Criterion | Points |
|---|-----------|--------|
| 4.11 | GET /api/suppliers — list all | 0.5 |
| 4.12 | GET /api/suppliers/{id} — detail with contacts | 1 |
| 4.13 | GET /api/suppliers/{id}/contacts — contacts only | 0.5 |
| 4.14 | POST /api/suppliers — create | 1 |
| 4.15 | PUT /api/suppliers/{id} — update | 1 |

### Purchase Orders (10 points)
| # | Criterion | Points |
|---|-----------|--------|
| 4.16 | GET /api/orders — list, filterable by status + supplier_id | 1 |
| 4.17 | GET /api/orders/{id} — includes line_items + status_history | 1.5 |
| 4.18 | GET /api/orders/workflow-stats — count per status | 1 |
| 4.19 | POST /api/orders — creates with line items, auto PO-YYYY-NNN, auto-calculates totals | 3 |
| 4.20 | PUT /api/orders/{id} — update notes | 0.5 |
| 4.21 | POST /api/orders/{id}/status — advances status, records history | 3 |

### Payments (6 points)
| # | Criterion | Points |
|---|-----------|--------|
| 4.22 | GET /api/payments — list, filterable by status | 0.5 |
| 4.23 | GET /api/payments/{id} — detail | 0.5 |
| 4.24 | GET /api/payments/summary — monthly totals, YTD, fees | 2 |
| 4.25 | POST /api/payments — create, auto PAY-YYYY-NNN | 1.5 |
| 4.26 | PUT /api/payments/{id} — update + audit log | 1 |
| 4.27 | GET /api/payments/{id}/sync-wise — stub | 0.5 |

### Inventory (12 points)
| # | Criterion | Points |
|---|-----------|--------|
| 4.28 | GET /api/inventory/ — list with computed status (OK/LOW/OUT_OF_STOCK) | 1.5 |
| 4.29 | GET /api/inventory/?status_filter=LOW — filter works | 1 |
| 4.30 | GET /api/inventory/stats — all 5 aggregate fields | 1.5 |
| 4.31 | GET /api/inventory/{product_id} — auto-creates if missing | 1 |
| 4.32 | GET /api/inventory/{product_id}/movements — paginated | 0.5 |
| 4.33 | POST /api/inventory/receive-shipment — bulk receive | 1.5 |
| 4.34 | POST /api/inventory/sale — single sale | 1.5 |
| 4.35 | POST /api/inventory/bulk-sale — atomic bulk | 1.5 |
| 4.36 | POST /api/inventory/adjust — absolute target, not delta | 1.5 |

### Dashboard (4 points)
| # | Criterion | Points |
|---|-----------|--------|
| 4.37 | GET /api/dashboard/ — 4 KPIs returned | 2 |
| 4.38 | GET /api/dashboard/order-pipeline — grouped by status | 1 |
| 4.39 | GET /api/dashboard/inventory-health — per-SKU summary | 1 |

---

## Section 5: Business Logic Correctness (25 points)

The hard stuff. Not "does the endpoint exist" but "does it do the right thing."

| # | Criterion | Points | How to Verify |
|---|-----------|--------|---------------|
| 5.1 | **PO status workflow enforces forward-only transitions** — cannot go STOCKED→DRAFT | 4 | POST invalid transition, expect 400/422 |
| 5.2 | **All 10 PO stages work in order** — can advance DRAFT→SENT→...→STOCKED | 3 | Walk through all 10 transitions |
| 5.3 | **Insufficient stock sale returns 400** — not 500, not silent failure | 3 | POST sale with qty > available |
| 5.4 | **Inventory status computed correctly** — OUT_OF_STOCK when qty=0, LOW when 0<qty<=reorder_point, OK when qty>reorder_point | 3 | Check status field on inventory list |
| 5.5 | **Atomic bulk operations** — receive-shipment and bulk-sale validate ALL items before modifying ANY; partial failure = full rollback | 4 | bulk-sale with one valid + one invalid item → nothing changed |
| 5.6 | **Adjustment uses absolute quantity** — posting qty=45 when current is 50 results in stock=45 and movement delta=-5 | 2 | POST adjust, verify inventory + movement |
| 5.7 | **Days until stockout formula** — velocity from 90-day sales, handles zero sales (infinity) | 2 | Inspect inventory_monitor logic or test output |
| 5.8 | **Alert deduplication** — same product doesn't trigger two alerts within 24h | 2 | Check notification records or test |
| 5.9 | **PO totals auto-calculated from line items** — subtotal = sum(qty * unit_price) | 2 | Create PO with items, check subtotal field |

---

## Section 6: Security (20 points)

Not "did you think about security" but "is the security actually implemented and working."

| # | Criterion | Points | How to Verify |
|---|-----------|--------|---------------|
| 6.1 | Passwords hashed with bcrypt (not plaintext, not MD5) | 3 | Inspect stored hash — starts with `$2b$` |
| 6.2 | Bcrypt work factor 12 | 1 | Hash contains `$2b$12$` |
| 6.3 | JWT access tokens expire in 15 minutes | 1.5 | Decode token, check `exp` claim |
| 6.4 | JWT uses HS256 | 0.5 | Decode token header |
| 6.5 | Refresh tokens with 7-day expiry | 1 | Decode or inspect config |
| 6.6 | TOTP MFA with pyotp | 2 | MFA setup returns valid TOTP URI |
| 6.7 | Rate limiting: auth returns 429 after 5 rapid attempts | 3 | Fire 6 rapid login requests |
| 6.8 | Global rate limiting: 100/min | 1 | Inspect middleware code |
| 6.9 | Security headers present (all 4: X-Content-Type-Options, X-Frame-Options, Referrer-Policy, CSP) | 2 | `curl -I /health`, check headers |
| 6.10 | CORS not set to wildcard `*` | 1 | Inspect main.py CORS config |
| 6.11 | Fernet encryption used for sensitive fields | 2 | Grep for Fernet in codebase |
| 6.12 | Bleach used for input sanitization | 1 | Grep for bleach import/usage |
| 6.13 | No raw SQL with user input anywhere in routers | 1 | Grep for `text(` in routers — should be zero |

---

## Section 7: Audit Trail (10 points)

| # | Criterion | Points | How to Verify |
|---|-----------|--------|---------------|
| 7.1 | audit_log records exist after CREATE operations | 2 | Create a product, query audit_log |
| 7.2 | audit_log records exist after UPDATE operations with old+new values | 2 | Update a product, check old_values/new_values |
| 7.3 | audit_log records exist after DELETE operations | 1 | Delete (soft) a product, check audit_log |
| 7.4 | password_hash and mfa_secret excluded from logged values | 2 | Register user, check audit_log — no sensitive fields |
| 7.5 | No-op updates don't create audit entries | 1.5 | PUT same values twice, second should not create entry |
| 7.6 | IP address and user-agent captured | 1.5 | Check audit_log fields after any operation |

---

## Section 8: Frontend (10 points)

| # | Criterion | Points | How to Verify |
|---|-----------|--------|---------------|
| 8.1 | Page renders 4 KPI cards | 3 | Visual inspection — Inventory Value, Pending Orders, Low Stock, MTD Revenue |
| 8.2 | Cards show real data from API (not hardcoded) | 2 | Change DB data, refresh page, values change |
| 8.3 | Responsive grid (1→2→4 columns at breakpoints) | 1.5 | Resize browser window |
| 8.4 | Loading state visible while fetching | 1 | Throttle network, observe spinner |
| 8.5 | Error state doesn't crash — shows zeroes or fallback | 1 | Kill backend, load frontend |
| 8.6 | TailwindCSS used (not inline styles or plain CSS) | 0.5 | Inspect source for Tailwind classes |
| 8.7 | Light theme, correct color palette (blue/green/amber/red) | 0.5 | Visual inspection |
| 8.8 | `layout.tsx` and `globals.css` exist and are configured | 0.5 | File check |

---

## Score Sheet

```
Section 1: Does It Run?              _____ / 30
Section 2: Test Suite                _____ / 35
Section 3: Schema Completeness       _____ / 20
Section 4: API Endpoints             _____ / 50
Section 5: Business Logic            _____ / 25
Section 6: Security                  _____ / 20
Section 7: Audit Trail               _____ / 10
Section 8: Frontend                  _____ / 10
                                    ============
TOTAL                                _____ / 200
```

---

## Grade Bands

| Score | Grade | What It Means |
|-------|-------|---------------|
| 180-200 | S | Exceptional. Ship it. |
| 160-179 | A | Production-quality with minor gaps |
| 140-159 | B | Solid work, some features incomplete or buggy |
| 120-139 | C | Core works, significant pieces missing |
| 100-119 | D | Partial implementation, many rough edges |
| 80-99 | E | Skeleton exists, limited functionality |
| Below 80 | F | Fundamentally incomplete |

---

## Automated Scoring Script Outline

For repeatable grading, run these checks programmatically:

```bash
#!/bin/bash
SCORE=0
BACKEND_DIR="webapp/backend"
FRONTEND_DIR="webapp/frontend"

echo "=== SECTION 1: DOES IT RUN? ==="

# 1.1 Backend starts
cd "$BACKEND_DIR"
timeout 10 uv run uvicorn app.main:app --port 18000 &
PID=$!
sleep 5
if kill -0 $PID 2>/dev/null; then
    echo "[+5] Backend starts"
    SCORE=$((SCORE+5))
else
    echo "[+0] Backend failed to start"
fi
kill $PID 2>/dev/null

# 1.2 Health check
HEALTH=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:8000/health)
if [ "$HEALTH" = "200" ]; then
    echo "[+3] Health check OK"
    SCORE=$((SCORE+3))
fi

# 2.2 Test count
TEST_COUNT=$(cd "$BACKEND_DIR" && uv run pytest --collect-only -q 2>/dev/null | tail -1 | grep -oP '\d+(?= test)')
echo "Tests collected: $TEST_COUNT"

# 2.3 Pass rate
RESULT=$(cd "$BACKEND_DIR" && uv run pytest -q 2>&1 | tail -1)
PASSED=$(echo "$RESULT" | grep -oP '\d+(?= passed)')
FAILED=$(echo "$RESULT" | grep -oP '\d+(?= failed)')
TOTAL=$((PASSED + FAILED))
if [ "$TOTAL" -gt 0 ]; then
    RATE=$((PASSED * 100 / TOTAL))
    echo "Pass rate: ${RATE}%"
fi

# 3.1 Table count
TABLE_COUNT=$(grep -c "class.*Base)" "$BACKEND_DIR/app/models.py" 2>/dev/null)
echo "Tables: $TABLE_COUNT"

# 6.7 Rate limiting
echo "=== RATE LIMIT TEST ==="
for i in {1..6}; do
    CODE=$(curl -s -o /dev/null -w "%{http_code}" -X POST http://localhost:8000/api/auth/login \
        -H "Content-Type: application/json" \
        -d '{"email":"x@x.com","password":"x"}')
    echo "Attempt $i: HTTP $CODE"
done

# 6.9 Security headers
echo "=== SECURITY HEADERS ==="
curl -sI http://localhost:8000/health | grep -iE "(x-content-type|x-frame|referrer-policy|content-security)"

echo "=== PARTIAL SCORE: $SCORE ==="
```

---

## Quick Reference: Point Distribution by Priority

| Priority | Sections | Points | % of Total |
|----------|----------|--------|------------|
| **Does it even work?** | Sec 1 (Run) + Sec 2 (Tests) | 65 | 32.5% |
| **Core functionality** | Sec 4 (APIs) + Sec 5 (Logic) | 75 | 37.5% |
| **Data integrity** | Sec 3 (Schema) + Sec 7 (Audit) | 30 | 15% |
| **Security** | Sec 6 | 20 | 10% |
| **Frontend** | Sec 8 | 10 | 5% |

This reflects what matters most: a backend that starts, passes its tests, and correctly implements the specified business logic is worth far more than a pretty frontend on a broken foundation.
