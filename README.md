# Biopharma Assessment — Backend

Node.js / Express backend scaffold using PostgreSQL, documented against the supplied Postman collections.

> **Implementation note:** The supplied materials define API requests and example request bodies, but do not include backend source code, an existing `package.json`, database schema, ORM choice, migration files, deployed base URL, or actual API response bodies. This README proposes a conventional Express + PostgreSQL setup. Treat commands as the intended baseline and adjust them to the implemented project. Do not claim migrations or tests have run until they exist and pass.

## 1. Technology

- Node.js (LTS)
- Express
- PostgreSQL
- `pg` (node-postgres) or a chosen ORM/query builder
- dotenv for local environment configuration
- Jest or Vitest + Supertest for API tests

The exact ORM and migration framework are not specified. If using raw SQL with `pg`, use a migration tool such as `node-pg-migrate`; if using Prisma/Sequelize/Knex, replace the migration commands with that tool's commands.

## 2. Prerequisites

- Node.js LTS and npm
- PostgreSQL server
- Git
- API application credentials (`APP_ID`, `APP_KEY`) and any upstream API credentials/tokens supplied by the API owner

## 3. Installation

```bash
git clone https://github.com/<GITHUB_OWNER>/biopharma-backend.git
cd biopharma-backend
npm install
```

## 4. Environment configuration

```bash
cp .env.example .env
```

Set the values for the local environment. Never commit `.env`.

## 5. PostgreSQL setup

Create a local database and user (adjust names/passwords to your environment):

```sql
CREATE USER biopharma_app WITH PASSWORD 'replace-with-a-local-secret';
CREATE DATABASE biopharma OWNER biopharma_app;
```

Verify the connection using `psql` or your preferred PostgreSQL client.

## 6. Database migrations

No schema or migration scripts were included in the supplied files. Therefore, no specific migration command or table structure can be confirmed from the API collections.

Recommended workflow if using `node-pg-migrate`:

```bash
npm run migrate:up
npm run migrate:down
```

Add migration files under `migrations/` and define the scripts in `package.json`, for example:

```json
{
  "scripts": {
    "migrate:up": "node-pg-migrate up",
    "migrate:down": "node-pg-migrate down"
  }
}
```

Do not run a rollback against shared/production data without a backup and approval.

## 7. Run

```bash
npm run dev
```

For production:

```bash
npm start
```

These scripts should be defined by the implementation. The recommended local API port is `3000`; change `PORT` if needed.

## 8. Build

Plain JavaScript Node.js projects generally do not require a compilation step:

```bash
npm run build
```

If using TypeScript, configure a compiler build (for example `tsc`) and ensure `npm start` runs the compiled output. The actual language configuration was not included in the assessment files.

## 9. Testing

Recommended commands (define the scripts and test suite before use):

```bash
npm test
npm run test:coverage
```

Use unit tests for services/controllers and integration tests with Supertest against a test database. Cover:
- Login, logout, token refresh and authorization.
- Valid and invalid/missing credentials.
- Requests, trackers and trackings CRUD operations.
- Lookup by ID, tracker code and request ID.
- Paging, sorting, offset/limit and search parameters.
- Validation, not-found, unauthorized and server-error behavior.
- Database constraints and transaction/rollback behavior.

Use a separate test database. Never run destructive tests against production.

## 10. API reference from the provided Postman collections

The Postman collection files are included in `docs/postman/`.

### Authentication collection

| Method | Path in collection | Notes |
|---|---|---|
| POST | `/auth/login` | `app_id` and `app_key` headers; JSON body contains `appCode`, `moduleCode`, `user`, `password` |
| POST | `/auth/logout` | Token-authenticated; `app_id` and `app_key`; body contains `appCode`, `moduleCode`, `user` |
| GET | `/auth/token/refresh` | `app_id` and `app_key`; OAuth2 refresh configuration in collection |
| GET | `/auth/authorization?nested=true` | Token-authenticated; `app_id` and `app_key` |

The collection-level API-key configuration uses header name `X-API-Key` and Postman variable `{{token}}`. Some requests also define OAuth2 token configuration. Confirm the actual server contract with the API owner; do not assume the Postman template fully describes token issuance or refresh semantics.

### Feature collection

| Resource | Methods / routes represented |
|---|---|
| Requests | `GET /requests`, `POST /requests`, `PUT /requests`, `DELETE /requests`, `GET /requests/:id`, `GET /requests/paging`, `GET /requests/paging/advance` |
| Trackers | `GET /trackers`, `POST /trackers`, `PUT /trackers`, `DELETE /trackers`, `GET /trackers/:id`, `GET /trackers/code/:code`, `GET /trackers/paging`, `GET /trackers/paging/advance` |
| Trackings | `GET /trackings`, `POST /trackings`, `GET /trackings/:id`, `GET /trackings/request-id/:requestId`, `GET /trackings/paging`, `GET /trackings/paging/advance` |

The exact route prefixes are represented by Postman variables (`{{BASE_URL}}`, `{{PATH_FEATURE}}`, `{{PATH_AUTHENTICATION}}`, and `{{PATH_AUTH}}`). Configure the actual values in Postman; the source files do not specify their deployed values.

### Request payload fields shown in the collection

- Requests create/update examples include `partnerId`, `projectCode`, `totalFee`, `lastStatus`, `receivingStatus`, `leadTime`, `receivedDate`, `analysisWipDate`, `analysisCompletedDate`, `id`, and audit fields such as `createdBy` / `updatedBy`; update also shows `isActive`.
- Trackers create/update examples include `code`, `parentCode`, `description`, `actionType`, `moduleFlow`, `isUsingNotification`, `isPublic`, `icon`, `order`, `mandays`, `isParallel`, `id`, audit fields, and `isActive` on update.
- Trackings create example includes `trackerCode`, `value`, `trackerId`, `requestId`, `partnerId`, `id`, and `createdBy`.

These are example bodies, not a confirmed database schema or exhaustive validation contract. The collection's saved example responses are empty or status-only (HTTP 200); response JSON schemas are not provided.

## 11. Suggested application structure

```text
src/
  app.js
  server.js
  config/
    database.js
  routes/
    auth.routes.js
    requests.routes.js
    trackers.routes.js
    trackings.routes.js
  controllers/
  services/
  repositories/
  middleware/
  validators/
  utils/
tests/
migrations/
docs/
  postman/
```

This is a suggested organization, not a description of source files already supplied.

## 12. Development workflow

1. Create a feature branch from `main`.
2. Update/author migration files when the schema changes.
3. Implement route → controller → service → repository layers.
4. Validate request payloads and authenticate/authorize protected routes.
5. Add unit and integration tests.
6. Run migrations against the local development database.
7. Run tests and lint checks.
8. Open a pull request; review before merging to `main`.

## 13. Security

- Store database passwords, `APP_KEY`, passwords and signing secrets only in environment variables or a secret manager.
- Never commit `.env`, credentials, live tokens, or real user data.
- Hash passwords if the application stores passwords (the supplied collection does not describe password storage).
- Use HTTPS in deployed environments.
- Validate inputs, parameterize SQL, enforce authorization server-side, and avoid logging secrets.
- Create a fine-grained GitHub token scoped only to this repository, with an expiry and minimum permissions. Share it securely, never in this README or a commit.
