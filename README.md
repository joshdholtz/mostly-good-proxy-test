# MGM Proxy

A tiny, blazing-fast reverse proxy for [Mostly Good Metrics](https://mostlygoodmetrics.com) that keeps your analytics working even when ad blockers are enabled.

[![Deploy to Fly.io](https://img.shields.io/badge/Deploy%20to-Fly.io-8b5cf6?style=for-the-badge&logo=fly.io)](https://fly.io/docs/speedrun/)
[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/template?code=mgm-proxy&referralCode=mostlygoodmetrics)
[![Deploy to Heroku](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/mostlygoodmetrics/mgm-proxy)
[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/mostlygoodmetrics/mgm-proxy)
[![Deploy to DO](https://www.deploytodo.com/do-btn-blue.svg)](https://cloud.digitalocean.com/apps/new?repo=https://github.com/mostlygoodmetrics/mgm-proxy/tree/main)

## Why use a proxy?

**Ad blockers and privacy extensions block analytics.** They maintain blocklists of known analytics domains, so requests to `ingest.mostlygoodmetrics.com` might never make it through.

**The fix?** Host this proxy on your own domain. Requests to `analytics.yourdomain.com` look like first-party traffic, so they don't get blocked.

```
Without proxy:
User → ingest.mostlygoodmetrics.com ← 🚫 Blocked by ad blocker

With proxy:
User → analytics.yourdomain.com → MGM
       ↳ Looks like your own API
       ↳ ✅ Not blocked!
```

## How it works

This proxy is dead simple:

1. Deploy it and point your own domain at it (e.g., `analytics.yourdomain.com`)
2. Your app sends events to your domain instead of MGM directly
3. The proxy forwards everything to MGM

No config, no database, no state. Just ~60 lines of Go.

## The stats

- **~5MB** Docker image
- **~10MB** memory usage
- **Zero** dependencies (Go standard library only)
- **<1ms** added latency

## Quick start

Pick your favorite platform and click a deploy button above, or keep reading for manual setup.

## Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `MGM_TARGET_URL` | `https://ingest.mostlygoodmetrics.com` | MGM ingestion endpoint |
| `PORT` | `8080` | Port to listen on |

## Deploy to Fly.io

Fly.io is great for this — deploy close to your users with edge regions.

```bash
# Install flyctl if needed
curl -L https://fly.io/install.sh | sh

# Clone and deploy
git clone https://github.com/mostlygoodmetrics/mgm-proxy.git
cd mgm-proxy
fly launch --no-deploy
fly deploy
```

Want it in a specific region? `fly deploy --region nrt` for Tokyo, `fly deploy --region fra` for Frankfurt, etc.

**Add a custom domain:**
```bash
fly certs add analytics.yourdomain.com
# Then add a CNAME record pointing to your-app.fly.dev
```

## Deploy to Heroku

```bash
heroku create my-mgm-proxy
heroku stack:set container
git push heroku main

# Add custom domain
heroku domains:add analytics.yourdomain.com
```

## Deploy to Digital Ocean

1. Click the "Deploy to DO" button above
2. Add your custom domain in the app settings
3. Done!

## Deploy to Railway

```bash
npm install -g @railway/cli
railway login
railway init
railway up

# Add custom domain in Railway dashboard
```

## Deploy to Render

1. Click the "Deploy to Render" button above
2. Add your custom domain in the service settings
3. Done!

## Run with Docker

```bash
docker run -p 8080:8080 ghcr.io/mostlygoodmetrics/mgm-proxy:latest
```

Or build it yourself:

```bash
docker build -t mgm-proxy .
docker run -p 8080:8080 mgm-proxy
```

## Run locally

```bash
go run main.go
# Listening on :8080
```

## Using the proxy

Once deployed with your custom domain, point your SDK to it:

**Swift (iOS/macOS)**
```swift
let config = MGMConfiguration(
    apiKey: "your-api-key",
    baseURL: URL(string: "https://analytics.yourdomain.com")!
)
MostlyGoodMetrics.configure(with: config)
```

**Kotlin (Android)**
```kotlin
val config = MGMConfiguration.Builder("your-api-key")
    .baseUrl("https://analytics.yourdomain.com")
    .build()
MostlyGoodMetrics.configure(this, config)
```

**JavaScript/TypeScript**
```javascript
import { MostlyGoodMetrics } from '@mostly-good-metrics/javascript';

MostlyGoodMetrics.configure({
    apiKey: 'your-api-key',
    baseURL: 'https://analytics.yourdomain.com',
});
```

## Health check

The proxy exposes a `/health` endpoint that returns `ok` with a 200 status. Use this for load balancer health checks.

```bash
curl https://analytics.yourdomain.com/health
# ok
```

## License

MIT — do whatever you want with it.
