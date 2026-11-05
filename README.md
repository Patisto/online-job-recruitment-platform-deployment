# Recruitment platform deployment

This repository runs the published backend image together with PostgreSQL and
private MinIO storage. It does not contain backend source code.

## Requirements

- Docker Engine
- Docker Compose v2
- Access to the backend image

## First installation

```bash
cp .env.example .env
```

Edit `.env` and replace every `REPLACE_*` value. Set
`APP_BOOTSTRAP_ADMIN_EMAIL` and `APP_BOOTSTRAP_ADMIN_PASSWORD` only when the
database is empty. The admin password must be 12–72 UTF-8 bytes.

Start the complete stack:

```bash
docker compose pull backend
docker compose up -d --build
docker compose ps
curl --fail http://localhost:8080/actuator/health
```

On the first successful startup, log in with the bootstrap admin account.
Then remove `APP_BOOTSTRAP_ADMIN_EMAIL` and
`APP_BOOTSTRAP_ADMIN_PASSWORD` from `.env` and recreate only the backend:

```bash
docker compose up -d --force-recreate backend
```

The admin account remains in PostgreSQL.

## Frontend API

The local API URL is:

```text
http://localhost:8080
```

Configure the frontend with its public API URL, for example:

```text
VITE_API_BASE_URL=http://localhost:8080
```

The browser must never receive PostgreSQL, JWT signing, MinIO root, SMTP, or
runtime database credentials. `PROFILE_STORAGE_PUBLIC_ENDPOINT` must be
reachable by the browser because the API returns short-lived private download
URLs for documents.

When `SPRING_PROFILES_ACTIVE=local`, documentation is available at:

```text
http://localhost:8080/swagger-ui/index.html
http://localhost:8080/v3/api-docs
```

## Updating the backend image

From the backend source repository, build and push a new immutable tag:

```bash
VERSION=1.0.1
IMAGE=ghcr.io/codeKaren777/online-job-recruitment-platform-backend

echo "$GHCR_TOKEN" | docker login ghcr.io -u codeKaren777 --password-stdin
docker build -t "$IMAGE:$VERSION" .
docker push "$IMAGE:$VERSION"
```

The GitHub token needs permission to write packages. Make the GHCR package
public if this deployment is intended to pull it without registry login.

On the deployment machine, update `.env`:

```env
BACKEND_IMAGE_TAG=1.0.1
```

Then deploy:

```bash
docker compose pull backend
docker compose up -d backend
docker compose ps
curl --fail http://localhost:8080/actuator/health
```

Flyway applies pending migrations automatically during backend startup.
Never reuse an old image tag for different backend contents.

## Stop and persistence

Stop containers without deleting data:

```bash
docker compose down
```

The `postgres_data` volume stores users and application data. The `minio_data`
volume stores uploaded profile documents and CVs. Do not use
`docker compose down -v` unless all data should be erased.

## Backups

Create a PostgreSQL backup:

```bash
docker compose exec -T db pg_dump \
  -U "$POSTGRES_USER" -d "$POSTGRES_DB" > recruitment-backup.sql
```

Back up MinIO objects separately. PostgreSQL backups do not contain uploaded
documents. Keep `.env`, database backups, and MinIO backups private.

## Fresh test installation

Warning: this permanently deletes the local PostgreSQL and MinIO volumes:

```bash
docker compose down -v
docker compose up -d --build
docker compose ps
curl --fail http://localhost:8080/actuator/health
```
