# StudyCenter

A role-based attendance management system for educational institutions. Admins manage students, teachers, and batches. Teachers mark and track attendance. Both roles get a shared dashboard with live stats.

## Stack

- **Frontend** — React 18, TypeScript, Vite, Tailwind CSS, shadcn/ui, TanStack Query, React Hook Form + Zod
- **Backend** — Node.js, Express, MongoDB (Mongoose)
- **Auth** — JWT (role-based: `admin` / `teacher`)

## Features

- **Admin** — full CRUD for students, teachers, and batches; dashboard stats; issue external API tokens
- **Teacher** — mark daily attendance per batch (bulk upsert); view batch/date attendance; monthly attendance view
- **External API** — scope-based JWT tokens for third-party integrations (`read` / `write`)
- Paginated attendance listing with batch and student filters

## Project Structure

```
studycenter/
├── backend/
│   ├── scripts/        # Seed scripts (admin, students)
│   └── src/
│       ├── controllers/
│       ├── middlewares/
│       ├── models/
│       ├── routes/
│       └── utils/
└── frontend/
    └── src/
        ├── components/
        ├── contexts/
        ├── hooks/
        ├── pages/
        └── types/
```

## Setup

### Requirements

- Node.js 18+
- MongoDB running locally

### Backend

```bash
cd backend
npm install
```

Create a `.env` file:

```env
MONGO_URI=mongodb://localhost:27017/attenl
JWT_SECRET=your_jwt_secret
EXTERNAL_JWT_SECRET=your_external_jwt_secret   # optional, falls back to JWT_SECRET
PORT=5000
```

Seed the default admin account:

```bash
node scripts/seedAdmin.js
# username: admin / password: password
```

Optionally seed dummy students (requires at least one batch to exist):

```bash
node scripts/seedStudents.js
```

Start the server:

```bash
npm run dev
```

### Frontend

```bash
cd frontend
npm install
npm run dev
```

Frontend runs at `http://localhost:5173` by default.

## API Overview

All routes (except `/api/auth/*`) require a `Bearer <token>` header.

| Method | Route | Role |
|--------|-------|------|
| POST | `/api/auth/login` | — |
| POST | `/api/auth/register` | — |
| GET | `/api/dashboard` | admin, teacher |
| GET/POST/PUT/DELETE | `/api/students` | admin |
| GET | `/api/students/:id/attendance` | admin |
| GET/POST/PUT/DELETE | `/api/teachers` | admin |
| GET/POST/PUT/DELETE | `/api/batches` | admin (write), teacher (read) |
| GET | `/api/attendance` | admin, teacher |
| POST | `/api/attendance/bulk-upsert` | teacher |
| GET | `/api/attendance/batch-date` | admin, teacher |
| GET | `/api/attendance/monthly` | admin, teacher |
| POST | `/api/external-auth/token` | admin |
| GET/POST | `/api/external/attendance/*` | external token |

## External API Tokens

Admins can issue scoped tokens for external integrations:

```http
POST /api/external-auth/token
Authorization: Bearer <admin_token>
Content-Type: application/json

{
  "scopes": ["read"],
  "expiresIn": "7d"
}
```

Use the returned token as a Bearer token against `/api/external/attendance/*` routes.
