# Self-Hosted Deployment (OpenResty + Astro SSR + Decap OAuth)

This project is configured for server deployment (not GitHub Pages).

## 1) Server requirements

- Linux server with a domain name, for example `blog.example.com`
- Node.js 20+
- pnpm 10+
- OpenResty (or Nginx)

## 2) GitHub OAuth app

Create an OAuth app at <https://github.com/settings/applications/new>

- Homepage URL: `https://blog.example.com`
- Authorization callback URL: `https://blog.example.com/oauth/callback`

Save:

- Client ID
- Client Secret

## 3) Update Decap config

Edit `public/admin/config.yml`:

- `site_domain`: your host only, no scheme (`blog.example.com`)
- `base_url`: full URL (`https://blog.example.com`)
- `auth_endpoint`: keep `oauth`

## 4) Set environment variables

Create `.env` (or set them in your process manager):

```bash
OAUTH_GITHUB_CLIENT_ID=xxxxx
OAUTH_GITHUB_CLIENT_SECRET=xxxxx
# Optional for GitHub App flow:
OAUTH_GITHUB_REPO_ID=
```

## 5) Build and run Astro server

```bash
pnpm install
pnpm build
pnpm start
```

The app will run with the Node standalone server (`dist/server/entry.mjs`).

## 6) OpenResty reverse proxy

Example server block:

```nginx
server {
    listen 80;
    server_name blog.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name blog.example.com;

    # Configure your cert files
    ssl_certificate /etc/letsencrypt/live/blog.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/blog.example.com/privkey.pem;

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
}
```

## 7) Verify

- Site: `https://blog.example.com/`
- Admin: `https://blog.example.com/admin`
- OAuth start: `https://blog.example.com/oauth`

If login is successful, Decap can commit content to your repo.
