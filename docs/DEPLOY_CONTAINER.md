# Container Deployment (Docker + OpenResty)

This project runs in SSR mode and can be deployed in a container.

## 1) Prepare `.env`

Copy from example:

```bash
cp .env.example .env
```

Set required OAuth vars in `.env`:

```bash
OAUTH_GITHUB_CLIENT_ID=your_client_id
OAUTH_GITHUB_CLIENT_SECRET=your_client_secret
# Optional:
OAUTH_GITHUB_REPO_ID=
```

## 2) Build and run container

```bash
docker compose build
docker compose up -d
docker compose logs -f mizuki
```

Service listens on `127.0.0.1:4321` (host local only).

## 3) OpenResty reverse proxy

Use your HTTPS vhost and proxy to container port:

```nginx
location / {
    proxy_pass http://127.0.0.1:4321;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

## 4) Verify

- `https://sweecidal.icu/`
- `https://sweecidal.icu/admin`
- `https://sweecidal.icu/oauth`

## 5) Update after new commits

```bash
git pull
docker compose build
docker compose up -d
```

## Notes

- Docker build uses `pnpm astro build` intentionally, not `pnpm build`.
- This avoids heavy extra tasks (`update-anime` + `compress-fonts`) that can
  cause high memory usage on small VPS instances.
