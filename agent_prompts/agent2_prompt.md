# Agent 2: Backend - Auth & Core

## Project Context
You are building a Small Business Inventory Management System. Tech stack: FastAPI (Python 3.11+), SQLAlchemy, PostgreSQL, Next.js 14.

## IMPORTANT: Fully Autonomous Operation
The user is NOT available until the project is complete to spec. You MUST:
- Make ALL decisions yourself or by coordinating with teammates via relay-mesh
- NEVER wait for user input — if you have questions, ask the team lead or relevant teammate via `send_message`
- Keep working until your deliverables are done and the team lead confirms completion
- Check `fetch_messages` regularly (after each deliverable, every few minutes while waiting)

## Your Responsibility
Build the core FastAPI application, authentication, and product/supplier routers.

## Deliverables

### 1. Configuration (`webapp/backend/app/config.py`)
Pydantic Settings class reading from env vars:
- DATABASE_URL, SECRET_KEY, DB_ENCRYPTION_KEY, ALGORITHM
- ACCESS_TOKEN_EXPIRE_MINUTES=15, REFRESH_TOKEN_EXPIRE_DAYS=7
- WISE_API_KEY, WISE_WEBHOOK_SECRET
- SMTP_HOST, SMTP_PORT, SMTP_USER, SMTP_PASSWORD, FROM_EMAIL
- DEBUG

### 2. Main Application (`webapp/backend/app/main.py`)
- FastAPI with lifespan manager for background tasks
- CORS: allow https://inventory.yourdomain.com and http://localhost:3000
- Global rate limiter: 100 req/min
- Auth endpoint rate limit: 5 req/min
- Security headers middleware
- Health check: GET /health → {"status": "ok"}

### 3. Auth Router (`webapp/backend/app/routers/auth.py`)
- POST /api/auth/register - Create user, hash password with bcrypt (work factor 12)
- POST /api/auth/login - Verify credentials, return JWT access + refresh tokens
- POST /api/auth/refresh - Exchange refresh token for new access token
- POST /api/auth/mfa/setup - Generate TOTP secret, return QR code URI
- POST /api/auth/mfa/verify - Validate TOTP code, enable MFA

JWT: HS256, 15-min access tokens, 7-day refresh tokens.

### 4. Products Router (`webapp/backend/app/routers/products.py`)
- GET /api/products - List (filter by best_seller, active)
- GET /api/products/{id} - Detail with supplier info
- POST /api/products - Create, record price in price_history
- PUT /api/products/{id} - Update; if price changed, record in price_history
- DELETE /api/products/{id} - Soft delete (is_active = False)

### 5. Suppliers Router (`webapp/backend/app/routers/suppliers.py`)
- GET /api/suppliers - List all
- GET /api/suppliers/{id} - Detail with contacts
- GET /api/suppliers/{id}/contacts - List contacts only
- POST /api/suppliers - Create
- PUT /api/suppliers/{id} - Update

## relay-mesh Registration

When registering with relay-mesh, use these **exact values**:
- `name`: "backend-auth"
- `project`: "inventory-management"
- `role`: "backend-engineer"
- `specialization`: "auth-api"
- `description`: "Backend engineer building authentication, products, and suppliers API"

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
- Report completion status via `broadcast_message`

## Dependencies
- Agent 1's models.py and schemas/
- Install: fastapi, uvicorn, sqlalchemy, pydantic-settings, python-jose, passlib, pyotp, slowapi

## Success Criteria
- All endpoints return correct status codes
- Auth rate limiting returns 429 after 5 attempts per minute
- Security headers present in responses
- GET /health returns 200
