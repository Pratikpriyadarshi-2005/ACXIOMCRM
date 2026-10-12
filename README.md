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