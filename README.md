# Backend Event Management System

En modern och säker REST API för hantering av events och användarregistreringar, byggd med Fastify, TypeScript och PostgreSQL.

## 📋 Innehållsförteckning

- [Översikt](#översikt)
- [Funktioner](#funktioner)
- [Teknikstack](#teknikstack)
- [Förutsättningar](#förutsättningar)
- [Installation](#installation)
- [Konfiguration](#konfiguration)
- [Köra projektet](#köra-projektet)
- [API Documentation](#api-dokumentation)
- [Databasstruktur](#databasstruktur)
- [Projektstruktur](#projektstruktur)
- [Säkerhet](#säkerhet)
- [Docker](#docker)
- [Testning](#testning)

## 🎯 Översikt

Detta projekt är en examination av backend-utveckling och implementerar en event-management-applikation. Systemet möjliggör:

- Användarregistrering och autentisering
- Skapande och hantering av events
- Bokning av events av registrerade användare
- Rollbaserad åtkomstkontroll (användare och admin)
- Säker API-kommunikation med JWT-tokens

## ✨ Funktioner

- **Autentisering & Autentisering**: JWT-baserad säkerhet med refresh-tokens
- **Event-management**: CRUD-operationer för events (begränsad till admins)
- **Användarregistreringar**: Bokning och avbokning av events
- **Rate Limiting**: Skydd mot brute-force-attacker
- **CORS-stöd**: Konfigurerad för att arbeta med frontend-applikationer
- **Säkra HTTP-headers**: Helmet-integration för säkerhet
- **Databasintegritet**: Relationskonstraint för att förhindra dubbelbokningar
- **Felhantering**: Centraliserad felhantering med standardiserade svar

## 🛠 Teknikstack

| Område | Teknologi |
|--------|-----------|
| **Runtime** | Bun |
| **Framework** | Fastify 5.8.2 |
| **Språk** | TypeScript |
| **Databas** | PostgreSQL 15 |
| **Autentisering** | JWT (@fastify/jwt) |
| **Säkerhet** | Helmet, CORS, bcrypt, rate-limiting |
| **Container** | Docker & Docker Compose |

## 📦 Förutsättningar

- Bun (senaste versionen)
- PostgreSQL 15+ eller Docker/Docker Compose
- Node.js 18+ (för kompatibilitet)
- Git

Installera Bun från: https://bun.sh

## 🚀 Installation

### 1. Klona repositoriet

```bash
git clone <repository-url>
cd Backend-examination
```

### 2. Installera beroenden

```bash
bun install
```

### 3. Skapa .env-fil

Skapa en `.env`-fil i projektets rotmapp:

```env
# Databaskonfiguration
DB_USER=postgres
DB_PASSWORD=postgres
DB_NAME=eventdb
DB_HOST=localhost
DB_PORT=5432

# JWT-hemligt
JWT_SECRET=din-superhemlinga-nyckel-123456789

# Miljö
NODE_ENV=development
```

## ⚙️ Konfiguration

### Databaskonfiguration

Databasen konfigureras i `src/config/db.ts`:

```typescript
import pool from './config/db';
// Använder pg Pool för anslutning
```

**Miljövariabler som behövs:**
- `DB_USER`: PostgreSQL-användarnamn
- `DB_PASSWORD`: PostgreSQL-lösenord
- `DB_NAME`: Databasens namn
- `DB_HOST`: Host för databasen (standard: localhost)
- `DB_PORT`: Port för PostgreSQL (standard: 5432)

### JWT-konfiguration

JWT-hemligt måste definieras i `.env`:

```env
JWT_SECRET=din-längtåständig-nyckel-med-många-tecken
```

**Tokens:**
- **Access Token**: Kortkörande token (valideras på protected routes)
- **Refresh Token**: Långvarig token för att få nya access-tokens

## 🏃 Köra projektet

### Utvecklingsläge (med hot-reload)

```bash
bun run dev
```

Servern startar på `http://localhost:3000`

### Produktion

```bash
bun run start
```

### Testa databasanslutning

```bash
bun run test-db
```

## 📡 API-dokumentation

### Health Check

```http
GET /api/health
```

**Respons:**
```json
{
  "status": "success",
  "db": "OK",
  "time": "2026-03-18T10:30:45.123Z"
}
```

### Autentisering

#### Registrera användare

```http
POST /api/auth/register
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securepwd123"
}
```

**Respons (201):**
```json
{
  "success": true,
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "role": "user"
  }
}
```

#### Logga in

```http
POST /api/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securepwd123"
}
```

**Respons (200):**
```json
{
  "success": true,
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
}
```

#### Förnya token

```http
POST /api/auth/refresh
Content-Type: application/json

{
  "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
}
```

### Events (Admin-only för POST, PUT, DELETE)

#### Hämta alla events

```http
GET /api/events
```

**Respons:**
```json
[
  {
    "id": "uuid",
    "title": "Webinarium TypeScript",
    "description": "Lär dig TypeScript",
    "event_date": "2026-04-15T14:00:00Z",
    "created_by": "uuid",
    "created_at": "2026-03-18T10:00:00Z"
  }
]
```

#### Skapa event (Admin)

```http
POST /api/events
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "title": "Meetup JavaScript",
  "description": "Nätverking för JS-utvecklare",
  "event_date": "2026-04-20T18:00:00Z"
}
```

#### Uppdatera event (Admin)

```http
PUT /api/events/:id
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "title": "Uppdaterad titel",
  "description": "Ny beskrivning",
  "event_date": "2026-04-25T19:00:00Z"
}
```

#### Radera event (Admin)

```http
DELETE /api/events/:id
Authorization: Bearer <accessToken>
```

### Registreringar (Bokningar)

#### Boka event

```http
POST /api/registrations
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "event_id": "uuid"
}
```

**Respons:**
```json
{
  "success": true,
  "registration": {
    "id": "uuid",
    "user_id": "uuid",
    "event_id": "uuid",
    "status": "pending"
  }
}
```

#### Hämta mina bokningar

```http
GET /api/users/me/registrations
Authorization: Bearer <accessToken>
```

#### Avboka event

```http
DELETE /api/registrations
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "event_id": "uuid"
}
```

## 🗄️ Databasstruktur

### Tabeller

**users** - Lagrar användarinformation
```sql
id (UUID) - Primärnyckel
email (VARCHAR) - Unik användaremejl
password_hash (TEXT) - Hashad lösenord (bcrypt)
role (VARCHAR) - Användarroll (user/admin)
refresh_token (TEXT) - Token för att förnya access-token
created_at (TIMESTAMP) - Registreringsdatum
```

**events** - Lagrar event-information
```sql
id (UUID) - Primärnyckel
title (VARCHAR) - Event-titel
description (TEXT) - Event-beskrivning
event_date (TIMESTAMP) - Eventets datum och tid
created_by (UUID) - Referens till skapare (users.id)
created_at (TIMESTAMP) - Skapningsdatum
```

**registrations** - Kopplar users till events (many-to-many)
```sql
id (UUID) - Primärnyckel
user_id (UUID) - Referens till användare (users.id)
event_id (UUID) - Referens till event (events.id)
status (VARCHAR) - Bokningsstatus (pending/confirmed/cancelled)
UNIQUE(user_id, event_id) - Förhindrar dubbelbokningar
```

## 📁 Projektstruktur

```
src/
├── app.ts                 # Huvudapplikation & Fastify-konfiguration
├── routes.ts              # API-routedefinitioner
├── config/
│   └── db.ts              # Databaskonfiguration
├── controllers/
│   ├── authController.ts  # Autentiseringslogik
│   ├── eventController.ts # Event-operationer
│   └── registrationController.ts # Bokningslogik
├── middleware/
│   └── auth.ts            # JWT-verifiering & rollkontroll
├── repositories/
│   ├── userRepository.ts  # Användarfrågor
│   ├── eventRepository.ts # Event-frågor
│   └── registrationRepository.ts # Bokningsfrågor
├── schemas/
│   └── validation.ts      # Zod/JSON-schemas för validering
└── types/
    └── fastify.d.ts       # TypeScript-definitioner
db/
└── init.sql               # Databasschemat & initiering
docker-compose.yml        # Docker-konfiguration
.env                      # Miljövariabler (ej i git)
package.json              # Beroenden & scripts
tsconfig.json             # TypeScript-konfiguration
```

### Arkitekturala mönster

**Repository Pattern**: Separerar databaslogik från business logic
**MVC-variant**: Controllers hanterar requests, repositories hanterar data
**Middleware-baserad säkerhet**: Centraliserad autentisering och auktorisering

## 🔐 Säkerhet

Projektet implementerar flera säkerhetsåtgärder:

### 1. Autentisering & Autentisering
- **JWT-tokens**: Stateless autentisering
- **Refresh-tokens**: Säkra långkörande sessions
- **bcrypt**: Krypterad lösenordslagring
- **Rollbaserad åtkontroll**: Admin-kontroll för känsliga operationer

### 2. Skydd mot attacker
- **Helmet**: Säkra HTTP-headers (CSP, X-Frame-Options, etc.)
- **Rate Limiting**: Max 100 requests per minut för att skydda mot brute-force
- **CORS**: Whitelist-baserad autentisering för frontend-domains
- **Unique constraints**: Förhindrar dubbelbokningar på databaskivå

### 3. Input-validering
- Zod/JSON-schemavalidering på alla endpoints
- Förhindrar SQL-injektion via parameteriserade frågor
- XSS-skydd genom validering av indata

### 4. Felhantering
- Centraliserad errorhandler avslöjar inte känslig information
- Loggning av fel för debugging

### Potentiella förbättringar

- SSL/TLS-kryptering för över-tråd-trafik
- HTTPS-enforcing i produktion
- Två-faktor autentisering (2FA)
- Audit-loggning för känsliga operationer
- API-nyckel-autentisering för tredjeparts-integreringar

## 🐳 Docker

### Docker Compose

Starta PostgreSQL med Docker Compose:

```bash
docker-compose up -d
```

Detta startar:
- PostgreSQL 15 på port 5432
- Initierar databasen med `db/init.sql`
- Lagrar data i Docker-volymer

**Miljövariabler läses från `.env`**

### Dockerfile för API (ej inkluderad, kan läggas till)

För att containerisera Fastify-appen:

```dockerfile
FROM oven/bun:latest

WORKDIR /app
COPY package.json bun.lockb ./
RUN bun install --frozen-lockfile

COPY . .
EXPOSE 3000
CMD ["bun", "src/app.ts"]
```

## 🧪 Testning

### Testa databasanslutning

```bash
bun run test-db
```

### Testa hälsoendpoint

```bash
curl http://localhost:3000/api/health
```

### Postman Collection

En Postman-collection skapas genom att importera filen i `postman/`:

```yaml
# globals/workspace.globals.yaml innehåller globala variabler
BASE_URL: http://localhost:3000
TOKEN: [uppdateras vid inloggning]
```

### Manuell testning

1. Registrera en användare
2. Logga in och spara `accessToken`
3. Testa protected endpoints med Bearer token

## 📚 Ytterligare resurser

- [Fastify Dokumentation](https://www.fastify.io/)
- [Bun Dokumentation](https://bun.sh/docs)
- [PostgreSQL Dokumentation](https://www.postgresql.org/docs/)
- [JWT.io](https://jwt.io/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)

## 📝 Anteckningar

- **Miljövariabler**: Läs från `.env` med `dotenv` - använd aldrig hårdkodade värden
- **Produktion**: Säkerställ `NODE_ENV=production` och uppdatera hemliga nycklar
- **Logging**: Fastify loggar alla requests i utvecklingsläge, disa i produktion för prestanda

## 📄 Licens

Detta projekt är en examensuppgift i backend-utveckling.

---

**Senast uppdaterad**: 18 mars 2026
