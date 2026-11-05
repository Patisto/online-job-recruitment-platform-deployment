# Frontend integration

This repository provides a Docker image deployment of the recruitment API,
PostgreSQL, and private MinIO document storage. The frontend only consumes the
HTTP API; it does not connect directly to PostgreSQL or MinIO.

## Start the API

Install Docker Engine and Docker Compose v2, then run:

```bash
cp .env.example .env
```

Set the `REPLACE_*` values in `.env`, then start the services:

```bash
docker compose pull backend
docker compose up -d --build
docker compose ps
curl --fail http://localhost:8080/actuator/health
```

The local API base URL is:

```text
http://localhost:8080
```

Configure the frontend:

```env
VITE_API_BASE_URL=http://localhost:8080
```

For another frontend build tool, use its public environment-variable
convention, such as `NEXT_PUBLIC_API_BASE_URL`.

## First administrator

On an empty database, set these backend-only values in `.env` before the first
startup:

```env
APP_BOOTSTRAP_ADMIN_EMAIL=admin@example.com
APP_BOOTSTRAP_ADMIN_PASSWORD=Use-a-strong-password
```

After the backend starts successfully, remove both values and recreate the
backend:

```bash
docker compose up -d --force-recreate backend
```

The admin account remains in PostgreSQL. Public registration creates
`JOB_SEEKER` accounts only.

## Authentication

1. Register with `POST /api/auth/register`.
2. Log in with `POST /api/auth/login`.
3. Send the returned token on protected requests:

```http
Authorization: Bearer <accessToken>
```

4. On `401`, clear the token and require login again.
5. Call `POST /api/auth/logout` when the user logs out.

Roles are `JOB_SEEKER`, `EMPLOYER`, and `ADMIN`. The API is authoritative for
roles, ownership, job expiry, and application status.

## Main user flow

1. Register and log in as a job seeker.
2. Create the profile before applying:
   `PUT /api/profile`.
3. Submit an employer upgrade request:
   `POST /api/employer-requests`.
4. An administrator approves the request.
5. Log in again to receive a token with the `EMPLOYER` role.
6. Create a company with `POST /api/companies`.
7. Create a vacancy with `POST /api/jobs`.
8. A job seeker browses jobs with `GET /api/jobs`.
9. The job seeker applies with multipart `jobId` and PDF `cv` using
   `POST /api/applications`.
10. The employer reviews applications and may shortlist or reject them.

The complete endpoint list, request bodies, response fields, roles, and status
codes is in [frontend-api-reference.md](frontend-api-reference.md).

## Profiles and documents

Create a profile before applying:

```json
{
  "displayName": "Jane Applicant",
  "fullName": "Jane Mary Applicant",
  "dateOfBirth": "1995-06-15",
  "phone": "+265999000111",
  "location": "Blantyre",
  "biography": "Backend developer"
}
```

Applications require a separate PDF CV upload, maximum 5 MiB. Profile documents
and application CVs are private. The API returns short-lived signed download
URLs; never construct storage URLs or access MinIO from the browser.

Employers can view applicant profile data only for applications belonging to
their own jobs. Employer application responses include the applicant profile,
and the dedicated profile route is:

```text
GET /api/employer/applications/{applicationId}/profile
```

## Browser and secrets

Only the API base URL belongs in frontend configuration. Never expose:

- PostgreSQL credentials or `DB_URL`
- `JWT_SECRET_BASE64`
- MinIO credentials
- SMTP credentials
- Admin bootstrap credentials

The API must be reachable by the browser through the configured origin,
development proxy, or approved CORS/reverse-proxy setup.

## Health and API documentation

Health check:

```text
GET http://localhost:8080/actuator/health
```

When the local profile is enabled:

```text
http://localhost:8080/swagger-ui/index.html
http://localhost:8080/v3/api-docs
```

## Data persistence

PostgreSQL data is stored in the `postgres_data` Docker volume. Uploaded
documents are stored in the `minio_data` Docker volume. Use:

```bash
docker compose down
```

to stop without deleting data. Do not use `docker compose down -v` unless all
local data should be erased.
