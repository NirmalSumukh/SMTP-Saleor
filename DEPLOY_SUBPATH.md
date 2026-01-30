# Deploying SMTP App on a Subpath (`/smtp`)

You have configured the app to run under the `/smtp` path. This is great for hosting everything under one domain (`saleor.leemasmart.com`).

## 1. App Configuration Changes
I have already updated `next.config.ts` with `basePath: "/smtp"`.
This means:
- The Manifest URL is now: `.../smtp/api/manifest`
- The Dashboard UI is at: `.../smtp/configuration`

## 2. Local Testing
When you run `pnpm run dev`:
- The app is now available at: `http://localhost:3010/smtp`
- **Important**: Trying to access `http://localhost:3010/` (root) will now give a 404. You MUST include `/smtp`.

## 3. Server Configuration (Reverse Proxy)

To make `saleor.leemasmart.com/smtp` work, you need to configure your web server (Nginx or Traefik) to route traffic.

### Option A: Nginx Configuration
Add this block to your Nginx server config for `saleor.leemasmart.com`:
```nginx
location /smtp {
    proxy_pass http://localhost:3010/smtp;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
}

location /_next/static {
    # If using Docker, you might need to alias this or just proxy it too
    proxy_pass http://localhost:3010/_next/static;
}
```

### Option B: Traefik (Docker Compose)
If you are using Docker Compose with Traefiklabels, add these labels to your **SMTP App** service in `docker-compose.yml`:

```yaml
services:
  smtp-app:
    image: saleor-app-smtp
    # ... other config ...
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.smtp-app.rule=Host(`saleor.leemasmart.com`) && PathPrefix(`/smtp`)"
      - "traefik.http.routers.smtp-app.entrypoints=websecure"
      - "traefik.http.routers.smtp-app.tls.certresolver=myresolver"
      - "traefik.http.services.smtp-app.loadbalancer.server.port=3010"
```

## 4. Installing in Saleor
Once deployed on your server:
1.  Go to **Saleor Dashboard**.
2.  Install App.
3.  Manifest URL: `https://saleor.leemasmart.com/smtp/api/manifest`

By using the same domain, you avoid all Cross-Origin (CORS) issues because the dashboard and the app share the same origin (`saleor.leemasmart.com`).
