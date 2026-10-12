# AcxiomCRM

Role-based sales CRM for customers, leads, opportunities, follow-ups, activities, and reporting. Sales staff see their own records. Managers see team pipeline. Administrators manage users, roles, and the audit trail.

The live application is deployed. Sign in with the demo account below.

## Live application

| | |
|---|---|
| Application | [https://acxiomcrm-murex.vercel.app](https://acxiomcrm-murex.vercel.app) |
| Sign in | [https://acxiomcrm-murex.vercel.app/login](https://acxiomcrm-murex.vercel.app/login) |
| API | [https://acxiomcrm-production.up.railway.app/api](https://acxiomcrm-production.up.railway.app/api) |
| Swagger UI | [https://acxiomcrm-production.up.railway.app/swagger](https://acxiomcrm-production.up.railway.app/swagger) |
| Health | [https://acxiomcrm-production.up.railway.app/health](https://acxiomcrm-production.up.railway.app/health) |

## Demo login

| | |
|---|---|
| Name | Administrator |
| Email | admin@acxiomcrm.com |
| Password | Acxiom#a443e854ebf6aA1 |
| Role | Admin |

Use the email and password on the sign-in page. Google sign-in is not connected. Public registration is disabled; an administrator creates staff accounts from Users.

After login the session lasts 30 minutes. Refreshing the page requires signing in again.

## What you can review

Sign in as Administrator, then walk the product in this order:

1. Open [the dashboard](https://acxiomcrm-murex.vercel.app/dashboard) while logged out. The app returns you to sign in.
2. Sign in and review KPI cards, lead-status, pipeline, and monthly sales charts. Change the date range.
3. Create a customer. Invalid email or phone is blocked in the form and again on the server. Phone numbers use international format, for example +919876543210.
4. Create a lead, set the status to Qualified, and convert it to a customer.
5. Create an opportunity. Amount must be greater than 0, probability must be from 0 to 100, and an open deal cannot close in the past. Weighted pipeline is amount × probability.
6. Schedule a follow-up. A new planned follow-up cannot be dated before today. Mark it completed or missed.
7. Open Reports for customer, lead, follow-up, opportunity, pipeline, and conversion views.
8. Open Users, Roles, and Audit Logs. These links appear only for Admin. Create a Manager or Sales Executive, then sign in as that user to see the narrower record scope.
9. Open [Swagger UI](https://acxiomcrm-production.up.railway.app/swagger) and call POST /api/auth/login, then the customer, lead, and opportunity endpoints with the returned bearer token.

## Roles

| Role | Access |
|---|---|
| Admin | All CRM records, users, roles, audit logs, and reports |
| Manager | Team customers, leads, opportunities, follow-ups, and reports |
| Sales Executive | Owned customers, leads, opportunities, follow-ups, and activities |

Authorization is enforced on the API. Hiding a menu link is not the access check.

## Product modules

- Dashboard — customers, leads, open / won / lost opportunities, pipeline value, and Chart.js charts
- Customers — create, edit, search, filter, and deactivate
- Leads — source, status, priority, assignment, and qualified-lead conversion
- Opportunities — stage, amount, probability, expected close date, and pipeline board
- Follow-ups — planned, completed, missed, and overdue
- Activities — call, meeting, email, and task
- Reports — operational and pipeline reports scoped to the signed-in role
- Administration — users, roles, account status, unlock, password reset, and audit log

## Security
- ASP.NET Core Identity hashes passwords. Plain-text passwords and hashes are never returned.
- Password policy: at least 8 characters, with uppercase, lowercase, a number, and a symbol.
- Five failed sign-ins lock the account for 15 minutes.
- Short-lived JWT bearer tokens. Logout updates the security stamp and revokes active sessions.
- Login, logout, lockout, create, update, delete, role changes, and password resets are audited.
- Requests are validated at the API. Duplicate customer email or phone, bad status transitions, and out-of-scope records are rejected.
- Unexpected failures return a safe message. Stack traces are not sent to the client.

## Architecture

Browser (Next.js on Vercel)
        │  HTTPS + bearer token
        ▼
ASP.NET Core API (Railway)
        │  Controllers → services → EF Core
        ▼
PostgreSQL (Supabase)

The browser never connects to the database. Identity owns users, roles, password hashes, and lockout. CRM writes and their audit rows are saved in one transaction. List and detail responses use explicit DTOs.

| Layer | Technology |
|---|---|
| UI | Next.js, React, TypeScript, Tailwind CSS, Chart.js |
| API | ASP.NET Core, ASP.NET Core Identity, JWT bearer auth, EF Core |
| Data | PostgreSQL on Supabase |
| Hosting | Vercel (web), Railway (API) |

## API

Interactive documentation: [Swagger UI](https://acxiomcrm-production.up.railway.app/swagger).

| Method | Endpoint | Purpose |
|---|---|---|
| POST | /api/auth/login | Sign in. Returns a bearer token. |
| POST | /api/auth/logout | Revoke the current session. |
| GET | /api/auth/me | Current user. |
| GET POST | /api/customers | List or create customers. |
| GET PUT DELETE | /api/customers/{id} | Read, update, or deactivate a customer. |
| GET POST | /api/leads | List or create leads. |
| GET POST | /api/opportunities | List or create opportunities. |
| GET POST | /api/followups | List or create follow-ups. |
| GET POST | /api/activities | List or create activities. |
| GET | /api/dashboard | KPI and chart data for the signed-in scope. |
| GET | /api/users | User administration. Admin only. |
| GET | /api/audit | Audit log. Admin only. |

Protected routes return 401 without a valid token and 403 for the wrong role. Validation failures return 400 or 422. Conflicts return 409.

To try an endpoint in Swagger: call POST /api/auth/login with the demo email and password, copy token, choose Authorize, and paste the token.

## Repository

frontend/crm-dashboard-ui-design   Next.js application
backend                            ASP.NET Core API
docs                               Architecture, deployment, and acceptance notes

## Run locally

Requirements: Node.js 22 or newer, and the .NET 10 SDK.

The deployed API is already available, so the UI can run against production:

cd frontend/crm-dashboard-ui-design
npm install
npm run dev

Set frontend/crm-dashboard-ui-design/.env.local:

NEXT_PUBLIC_API_URL=https://acxiomcrm-production.up.railway.app/api

Open [http://localhost:3000/login](http://localhost:3000/login) and use the demo account.

To run the API yourself, configure backend/.env from backend/.env.example, apply EF Core migrations, bootstrap the first admin, then start the API:

cd backend
dotnet run -- --bootstrap
dotnet run --urls http://localhost:5000

Point NEXT_PUBLIC_API_URL at http://localhost:5000/api and restart the Next.js dev server. Bootstrap reads Bootstrap__Email and Bootstrap__Password and creates that user as Admin once.

## Further reading

- [docs/architecture.md](docs/architecture.md)
- [docs/deployment.md](docs/deployment.md)
- [docs/acceptance.md](docs/acceptance.md)
- [docs/assignment.md](docs/assignment.md)