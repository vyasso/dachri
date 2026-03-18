# Backend  System

En REST API för event-management bygd med Fastify, TypeScript och PostgreSQL.

## 📋 Innehåll

- [Översikt](#översikt)
- [Installation](#installation)
- [Köra projektet](#köra-projektet)
- [API](#api)
- [Docker](#docker)
- [Postman](#postman)
- [Projektstruktur](#projektstruktur)

## 🎯 Översikt

En examination inom backend-utveckling som implementerar ett event-management system med:
- Användarregistrering och JWT-autentisering
- Event CRUD (admin-only)
- Event-bokningar för användare
- Rollbaserad åtkomstkontroll

**Tech Stack:** Bun • Fastify • TypeScript • PostgreSQL • Docker

## 🚀 Installation

### Förutsättningar
- [Bun](https://bun.sh) eller Node.js 18+
- PostgreSQL (eller använd Docker)

### Setup

```bash
# Klona och installera
git clone <https://github.com/vyasso/Backend-examination>
cd Backend-examination
bun install

# Skapa .env-fil
echo "DB_USER=postgres
DB_PASSWORD=postgres
DB_NAME=eventdb
DB_HOST=localhost
DB_PORT=5432
JWT_SECRET=din-hemlig-nyckel-här" > .env
```

## 🏃 Köra projektet

```bash
# Utveckling (hot-reload)
bun run dev

# Produktion
bun run start

# Testa databaskoppling
bun run test-db
```

Servern körs på `http://localhost:3000`

## 📡 API

### Autentisering

```bash
# Registrera
POST /api/auth/register
{ "email": "user@example.com", "password": "pwd123" }

# Logga in
POST /api/auth/login
{ "email": "user@example.com", "password": "pwd123" }

# Förnya token
POST /api/auth/refresh
{ "refreshToken": "..." }
```

### Events (admin-only för POST/PUT/DELETE)

```bash
GET /api/events                    # Hämta alla
POST /api/events                   # Skapa (admin)
PUT /api/events/:id                # Uppdatera (admin)
DELETE /api/events/:id             # Radera (admin)
```

### Registreringar

```bash
POST /api/registrations            # Boka event
GET /api/users/me/registrations    # Mina bokningar
DELETE /api/registrations          # Avboka
```

## 📁 Projektstruktur

```
src/
├── app.ts                      # Fastify-setup
├── routes.ts                   # API-routes
├── config/db.ts                # Databaskonfiguration
├── controllers/                # API-logik
├── middleware/auth.ts          # JWT & roller
├── repositories/               # Databasförfrågningar
├── schemas/validation.ts       # Input-validering
└── types/fastify.d.ts          # TypeScript-typer
db/init.sql                    # Databasschema
docker-compose.yml             # Docker-setup
```

**Arkitektur:** Repository Pattern + MVC

## Postman

Kollektion finns i `postman/`:
- Globala variabler: `postman/globals/workspace.globals.yaml`
- Importera för att testa alla endpoints

Miljövariabler uppdateras automatiskt vid inloggning.

## Säkerhet

- **JWT-autentisering** med access & refresh tokens
- **bcrypt** för lösenord
- **Helmet** för säkra headers
- **Rate limiting**: 100 req/min
- **CORS** & input-validering
- **Unika constraints** förhindrar dubbelbokningar

## Notes

- `.env` behövs för databas & JWT-hemligt
- `bun run dev` startar med hot-reload
- `bun run test-db` kontrollerar databasanslutning

## Docker

Redan konfigurerad med PostgreSQL i `docker-compose.yml`:

```bash
# Starta databasen
docker-compose up -d

# Stopp
docker-compose down
```

Miljövariabler läses från `.env`
