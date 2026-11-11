# Recruitment API

This package provides the recruitment backend API together with PostgreSQL
and private MinIO storage.

## Use the API

```bash
cp .env.example .env
```

Fill in the `REPLACE_*` values in `.env`, then pull and start the package:

```bash
docker compose pull
docker compose up -d --build
curl --fail http://localhost:8080/actuator/health
```

The backend image is downloaded from the public GitHub Container Registry.
This deployment repository does not contain backend source code.

The API base URL is:

```text
http://localhost:8080
```

Configure the frontend with:

```env
VITE_API_BASE_URL=http://localhost:8080
```

For all setup, authentication, user flows, file handling, and security
guidance, read [frontend-integration.md](frontend-integration.md).

For every endpoint, request body, response shape, role, and status code, read
[frontend-api-reference.md](frontend-api-reference.md).

## Important

- The browser communicates only with the API.
- Do not expose `.env` or any backend credentials to the browser.
- Do not connect the frontend directly to PostgreSQL or MinIO.
- Use `docker compose down` to stop without deleting data.
- `docker compose down -v` deletes the local database and uploaded documents.
