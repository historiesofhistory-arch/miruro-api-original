# Deployment Guide — Render / Railway / Fly.io / Koyeb

The Docker image is automatically built and published to GitHub Container Registry on every push to `main`.

## Image URL

```
ghcr.io/historiesofhistory-arch/miruro-api-original:latest
```

Public — no auth needed to pull. Available tags:
- `latest` — most recent build from `main`
- `main` — alias of `latest`
- `sha-<commit>` — pinned to a specific commit (e.g. `sha-6a74337`)

---

## Deploy on Render

1. Go to https://render.com → **New** → **Web Service**
2. Select **"Deploy an existing image from a registry"** (not "Connect a repository")
3. Image URL: `ghcr.io/historiesofhistory-arch/miruro-api-original:latest`
4. Region: closest to your users
5. Instance type: any (the API is lightweight — 512MB RAM is plenty)
6. Set environment variables:
   - `PORT` = `8000` (Render auto-detects, but set it explicitly)
   - `NODE_ENV` = `production`
   - `SESSION_SECRET` = any random 32+ char string
   - `DATABASE_URL` = optional — only if you wire up a DB later (not required for episodes/sources)
7. Health check path: `/` (returns 200)
8. Click **Create Web Service**

Render will pull the image and start it. The API will be live at `https://<your-service>.onrender.com/`.

---

## Deploy on Railway

1. Go to https://railway.app → **New Project** → **Deploy from Docker image**
2. Image: `ghcr.io/historiesofhistory-arch/miruro-api-original:latest`
3. Add env vars (same as Render)
4. Railway auto-detects port from `PORT` env var
5. Deploy

---

## Deploy on Fly.io

```bash
fly launch --image ghcr.io/historiesofhistory-arch/miruro-api-original:latest
fly secrets set PORT=8000 NODE_ENV=production SESSION_SECRET=your-secret
fly deploy
```

---

## Deploy on Koyeb

1. Go to https://app.koyeb.com → **Create Service**
2. Choose **Docker image**
3. Image: `ghcr.io/historiesofhistory-arch/miruro-api-original:latest`
4. Set env vars
5. Exposed port: 8000
6. Path: `/`
7. Deploy

---

## Verify deployment

After deploy, open `https://<your-domain>/` — you should see the API documentation homepage. Test endpoints:

```bash
# Should return trending anime
curl https://<your-domain>/trending

# Should return search results
curl "https://<your-domain>/search?query=naruto"

# Should return anime details
curl https://<your-domain>/info/20
```

The `/episodes/{id}` and `/watch/{...}` endpoints hit Cloudflare-protected miruro.tv — those will return 403 on datacenter IPs (Render, Railway, etc.). The README documents this; use the CF bypass panel in the modified version (`miruro-api-cf` repo) or run on a residential-IP VPS for those endpoints.

---

## Update the image

Any push to `main` triggers a rebuild. The `latest` tag updates automatically. Render/Railway/Fly will auto-redeploy if you enable "Auto-deploy on image update".

---

## Rebuild manually

Go to https://github.com/historiesofhistory-arch/miruro-api-original/actions → "Build and Publish Docker Image" → "Run workflow".
