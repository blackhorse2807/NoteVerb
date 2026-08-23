# NotesVerb — Microservices Note-Taking Backend

**Author:** Ujjwal Kumar Singh

A scalable note-taking backend built with a microservices architecture using Node.js, TypeScript, Express, PostgreSQL, and Prisma. Designed as a portfolio project to demonstrate distributed systems, secure REST APIs, and production-oriented backend practices.

## Architecture Overview

| Service | Port | Responsibility |
|---------|------|----------------|
| **API Gateway** | 8080 | Central entry point, reverse-proxy routing, JWT gate |
| **Auth Service** | 3001 | Registration, login, JWT access/refresh tokens |
| **User Service** | 3002 | User profile management |
| **Notes Service** | 3003 | Notes CRUD, soft delete, search, pagination |
| **Tags Service** | 3004 | Tag management and validation |

Each service owns its own PostgreSQL database (**database-per-service**). Shared types, middleware, and utilities live in `shared/`.

## Tech Stack

- **Runtime:** Node.js (v18+) with TypeScript
- **Framework:** Express.js
- **Database:** PostgreSQL with Prisma ORM
- **Auth:** JWT (access + refresh), bcrypt
- **Validation / Security:** Joi, Helmet, CORS, input sanitization
- **Gateway:** http-proxy-middleware
- **Containers:** Docker & Docker Compose (PostgreSQL)
- **Testing:** Jest

## Prerequisites

- Node.js v18 or higher
- Docker and Docker Compose
- Git

## Quick Start

### 1. Clone the repository

```bash
git clone <your-repo-url>
cd notesverb-yt-main
```

### 2. Environment setup

```bash
cp .env.example .env
cp api-gateway/.env.example api-gateway/.env
cp services/auth-service/.env.example services/auth-service/.env
cp services/user-service/.env.example services/user-service/.env
cp services/notes-service/.env.example services/notes-service/.env
cp services/tags-service/.env.example services/tags-service/.env
```

Update critical values in each `.env`:

- `JWT_SECRET` and `JWT_REFRESH_SECRET` (strong, unique secrets)
- `DATABASE_URL` (or use the Docker defaults below)

```bash
openssl rand -base64 32
```

### 3. Start PostgreSQL

```bash
docker-compose up postgres -d
```

This creates four databases via `init-database.sql`: `notesverb_auth`, `notesverb_users`, `notesverb_notes`, `notesverb_tags`.

### 4. Install dependencies

```bash
cd api-gateway && npm install
cd ../services/auth-service && npm install
cd ../user-service && npm install
cd ../notes-service && npm install
cd ../tags-service && npm install
cd ../../shared && npm install
```

### 5. Run Prisma migrations

```bash
cd services/auth-service && npx prisma migrate dev && npx prisma generate
cd ../user-service && npx prisma migrate dev && npx prisma generate
cd ../notes-service && npx prisma migrate dev && npx prisma generate
cd ../tags-service && npx prisma migrate dev && npx prisma generate
```

### 6. Start services (one terminal each)

```bash
# Auth
cd services/auth-service && npm run dev

# User
cd services/user-service && npm run dev

# Notes
cd services/notes-service && npm run dev

# Tags
cd services/tags-service && npm run dev

# API Gateway
cd api-gateway && npm run dev
```

Gateway URL: `http://localhost:8080`

## API Endpoints

All client traffic goes through the API Gateway (`/api/...`).

### Auth

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/register` | Register |
| POST | `/api/auth/login` | Login |
| POST | `/api/auth/refresh` | Refresh tokens |
| POST | `/api/auth/logout` | Logout |

### Users

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/users/profile` | Get profile |
| PUT | `/api/users/profile` | Update profile |

### Notes

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/notes` | Create note |
| GET | `/api/notes` | List notes (pagination / search) |
| GET | `/api/notes/:id` | Get note by id |

### Tags

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/tags` | Create tag |
| GET | `/api/tags` | List tags |

Protected routes require: `Authorization: Bearer <access_token>`.

## Testing

```bash
cd services/auth-service && npm test
cd ../user-service && npm test
cd ../notes-service && npm test
```

## Project Structure

```
notesverb/
├── api-gateway/              # Gateway (auth gate + proxy)
├── services/
│   ├── auth-service/         # Auth + JWT
│   ├── user-service/         # Profiles
│   ├── notes-service/        # Notes
│   └── tags-service/         # Tags
├── shared/                   # Shared types, middleware, utils
├── docker-compose.yml        # PostgreSQL
├── init-database.sql         # Per-service DB bootstrap
└── README.md
```

## Security Notes

1. Never commit `.env` files
2. Use strong, unique JWT secrets in production
3. Keep `JWT_SECRET` consistent across gateway and services
4. Restrict `CORS_ORIGIN` to your frontend domain
5. Prefer HTTPS in production

## Health Checks

| Service | URL |
|---------|-----|
| API Gateway | `http://localhost:8080/health` |
| Auth | `http://localhost:3001/health` |
| User | `http://localhost:3002/health` |
| Notes | `http://localhost:3003/health` |
| Tags | `http://localhost:3004/health` |

## Troubleshooting

1. **Port conflicts** — free ports `8080` and `3001–3004`
2. **DB connection** — confirm Postgres is up and `DATABASE_URL` matches Docker credentials
3. **JWT errors** — align `JWT_SECRET` across all services
4. **CORS issues** — set `CORS_ORIGIN` to your client URL

## License

MIT
