
# URL Shortener – NestJS + TypeORM + PostgreSQL
A **step-by-step, agent-ready build guide**

---

## 1. Project Overview

**Goal**
Build a production-grade URL shortener API using NestJS with a clean, module-based architecture.

**Core Features**
- Create short URLs
- Auto-generate short codes when alias not provided
- Custom aliases (e.g. `/hello`)
- URL redirection
- Expiry date support
- Click counter
- Analytics endpoint

**Non-goals (for v1)**
- Authentication
- UI / dashboard
- Redis / caching
- Microservices

---

## 2. Tech Stack (Locked)

- Node.js (>=18)
- NestJS
- TypeORM
- PostgreSQL
- class-validator / class-transformer
- nanoid

⚠️ **DO NOT** replace TypeORM with Prisma or Sequelize  
⚠️ **DO NOT** add frontend code

---

## 3. Folder & Module Architecture

```
src/
 ├── app.module.ts
 ├── main.ts
 ├── database/
 │    └── typeorm.config.ts
 ├── url/
 │    ├── url.module.ts
 │    ├── url.controller.ts
 │    ├── url.service.ts
 │    ├── url.entity.ts
 │    └── dto/
 │         ├── create-url.dto.ts
 │         └── analytics.dto.ts
 └── common/
      └── utils/
           └── code-generator.ts
```

**RULE**
- Controllers handle HTTP
- Services handle business logic
- Entities map DB schema
- DTOs define API contracts

---

## 4. Step-by-Step Build Guide

### Step 1: Bootstrap Project

```bash
npm i -g @nestjs/cli
nest new url-shortener
cd url-shortener
```

Choose **npm** or **pnpm**, not both.

---

### Step 2: Install Dependencies

```bash
npm install @nestjs/typeorm typeorm pg
npm install class-validator class-transformer nanoid
```

---

### Step 3: Configure PostgreSQL

Create database manually:

```sql
CREATE DATABASE url_shortener;
```

Create `typeorm.config.ts`:

```ts
export const typeOrmConfig = {
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'postgres',
  password: 'password',
  database: 'url_shortener',
  autoLoadEntities: true,
  synchronize: true, // ONLY for local dev
};
```

⚠️ DO NOT use `synchronize: true` in production

---

### Step 4: URL Entity Design

Rules:
- `shortCode` must be UNIQUE
- No separate `customAlias` field
- `clickCount` stored, not calculated

---

### Step 5: DTO Validation Rules

- `originalUrl` → valid URL
- `alias` → optional, max 30 chars
- Alias allowed chars: `a-z A-Z 0-9 - _`

---

### Step 6: Short Code Logic

Rules:
- If alias provided → use it
- Else → auto-generate (nanoid, length 7)
- Always check uniqueness before saving

---

### Step 7: Create Short URL Endpoint

```
POST /urls
```

Request:
```json
{
  "originalUrl": "https://hello.com/test",
  "alias": "hello",
  "expiresAt": "2026-01-01T00:00:00Z"
}
```

Response:
```json
{
  "shortUrl": "short.link/hello"
}
```

---

### Step 8: Redirect Endpoint

```
GET /:code
```

Logic order:
1. Find by shortCode
2. If not found → 404
3. If expired → 410
4. Increment clickCount
5. Redirect (302)

---

### Step 9: Analytics Endpoint

```
GET /urls/:code/analytics
```

Response:
```json
{
  "originalUrl": "...",
  "shortCode": "hello",
  "clickCount": 123,
  "createdAt": "...",
  "expiresAt": "..."
}
```

Read-only. No mutations.

---

## 5. Reserved Alias Rules

Block these aliases:
```
urls
health
analytics
admin
```
Always validate before saving.

---

## 6. Database Rules (IMPORTANT)

DO:
- Index shortCode
- Use UUID as primary key
- Use atomic increment for click count

DON'T:
- Query by original URL for redirect
- Calculate click counts dynamically
- Store analytics in JSON blobs

---

## 7. Error Handling Rules

| Case | Status |
|----|------|
| Alias exists | 409 |
| URL not found | 404 |
| URL expired | 410 |
| Invalid input | 400 |

---

## 8. Dos and Don’ts

### ✅ DO
- Write clean services
- Validate all input
- Keep modules isolated
- Commit frequently

### ❌ DON'T
- Add auth in v1
- Add frontend
- Use global state
- Over-engineer

---

## 9. Testing Expectations

Minimum:
- Service unit tests
- Alias conflict test
- Expiry logic test

Optional:
- E2E redirect test

---

## 10. Completion Checklist

- [ ] Project boots without errors
- [ ] Short URL creation works
- [ ] Custom alias works
- [ ] Redirect works
- [ ] Expiry enforced
- [ ] Click count increments
- [ ] Analytics endpoint works

---

## 11. Evaluation Criteria (for reviewers / agents)

- Code clarity
- Proper NestJS patterns
- Correct DB design
- No unnecessary features

---

## 12. Final Note

This is NOT a toy project.
Build it as if another engineer will maintain it after you.

