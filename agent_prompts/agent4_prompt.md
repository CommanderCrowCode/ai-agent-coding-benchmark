# Agent 4: Frontend

## Project Context
You are building the frontend for a Small Business Inventory Management System. Tech stack: Next.js 14 with React 18 and TailwindCSS.

## IMPORTANT: Fully Autonomous Operation
The user is NOT available until the project is complete to spec. You MUST:
- Make ALL decisions yourself or by coordinating with teammates via relay-mesh
- NEVER wait for user input — if you have questions, ask the team lead or relevant teammate via `send_message`
- Keep working until your deliverables are done and the team lead confirms completion
- Check `fetch_messages` regularly (after each deliverable, every few minutes while waiting)

## Your Responsibility
Create the Next.js frontend with dashboard page.

## Deliverables

### 1. Project Setup
- Initialize Next.js 14 project in `webapp/frontend/`
- Configure TypeScript, TailwindCSS
- Set up API client (use fetch or axios)

### 2. Configuration Files
- `package.json` - Dependencies
- `tsconfig.json` - TypeScript config
- `tailwind.config.ts` - Tailwind configuration (light theme)
- `next.config.js` - Next.js config

### 3. Dashboard Page (`webapp/frontend/src/app/page.tsx`)
Client component ('use client'):
- Fetch GET /api/dashboard/ on mount
- 4 KPI cards in responsive grid (1→2→4 columns):
  - Total Inventory Value (USD)
  - Pending Orders
  - Low Stock Alerts
  - Month-to-Date Revenue
- 2 sections below: Order Pipeline, Inventory Health (placeholder)
- Loading spinner state, error fallback with zeroes

### 4. Styling
- TailwindCSS throughout
- Light theme only (white background, dark text)
- Color palette: Blue primary, Green success, Amber warning, Red danger

### 5. Layout (`webapp/frontend/src/app/layout.tsx`)
- Root layout with proper metadata

### 6. Global Styles (`webapp/frontend/src/app/globals.css`)
- TailwindCSS imports

## relay-mesh Registration

When registering with relay-mesh, use these **exact values**:
- `name`: "frontend-dev"
- `project`: "inventory-management"
- `role`: "frontend-engineer"
- `specialization`: "nextjs-react"
- `description`: "Frontend engineer building Next.js dashboard and UI"

### After registration completes
1. Call `list_agents` to discover all registered teammates
2. Call `send_message` to introduce yourself to the team lead (look for role "team-lead"). If no lead found yet, call `broadcast_message` instead
3. Call `fetch_messages` to check if anyone has already sent you work or instructions

### Continuous Coordination
- After completing each deliverable, call `fetch_messages` and report progress to the team lead via `send_message`
- If you have questions or blockers, ask the team lead or relevant teammate — do NOT wait for user input
- When all your deliverables are done, send a completion message to the team lead listing what you built
- Keep checking `fetch_messages` even after your work is done — the team lead may ask for fixes or changes
- Coordinate with devops-testing agent for Docker integration
- Report completion status via `broadcast_message`

## Dependencies
- next >= 14
- react >= 18
- tailwindcss

## Success Criteria
- Dashboard loads and displays 4 KPI cards
- Responsive layout works on mobile/desktop
- Loading and error states handled
