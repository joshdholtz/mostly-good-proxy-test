# MGM Proxy

A tiny, blazing-fast reverse proxy for [Mostly Good Metrics](https://mostlygoodmetrics.com) that makes sure we know where your users actually are.

[![Deploy to Fly.io](https://img.shields.io/badge/Deploy%20to-Fly.io-8b5cf6?style=for-the-badge&logo=fly.io)](https://fly.io/docs/speedrun/)
[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/template?code=mgm-proxy&referralCode=mostlygoodmetrics)
[![Deploy to Heroku](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/mostlygoodmetrics/mgm-proxy)
[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/mostlygoodmetrics/mgm-proxy)
[![Deploy to DO](https://www.deploytodo.com/do-btn-blue.svg)](https://cloud.digitalocean.com/apps/new?repo=https://github.com/mostlygoodmetrics/mgm-proxy/tree/main)

## Do I need this?

**Maybe not!** If you're sending events directly from a mobile app or web browser to MGM, you're all set. We automatically detect your users' locations from their IP addresses.

**But if you're sending events through a server** (like a Next.js API route, a serverless function, or your own backend), then yes — you probably want this proxy.

### The problem

When events flow through your server, they arrive at MGM with *your server's* IP address, not your user's. So instead of seeing users from Tokyo, Berlin, and São Paulo, you see everyone coming from `us-east-1`. Not super useful.

```
Without proxy:
User (Tokyo) → Your Server (Virginia) → MGM
                                         ↳ "Oh cool, another user from Virginia"

With proxy:
User (Tokyo) → Your Server (Virginia) → MGM Proxy → MGM
                                         ↳ Extracts real IP
                                         ↳ "User from Tokyo!"
```

### When you need this

- Sending events from **Next.js API routes** or **server components**
- Using **serverless functions** (AWS Lambda, Vercel, Cloudflare Workers)
- Proxying through your **own backend** for security
- Running behind **Cloudflare** or another CDN that masks IPs

### When you don't need this

- Sending events **directly from iOS/Android apps** to MGM
- Sending events **directly from web browsers** to MGM
- Your server already forwards `X-Forwarded-For` headers (rare)

## How it works

This proxy is dead simple. It:

1. Receives your event request
2. Extracts the real client IP from headers like `CF-Connecting-IP`, `X-Forwarded-For`, etc.
3. Forwards the request to MGM with the real IP attached
4. Returns the response

That's it. No config, no database, no state. Just ~60 lines of Go.

## The stats

- **~5MB** Docker image
- **~10MB** memory usage
- **Zero** dependencies (Go standard library only)
- **<1ms** added latency

## Quick start

Pick your favorite platform and click the button above, or keep reading for manual setup.

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

## Deploy to Heroku

```bash
heroku create my-mgm-proxy
heroku stack:set container
git push heroku main
```

## Deploy to Digital Ocean

1. Click the "Deploy to DO" button above
2. That's it. Seriously.

## Deploy to Railway

```bash
npm install -g @railway/cli
railway login
railway init
railway up
```

## Deploy to Render

1. Click the "Deploy to Render" button above
2. Done!

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

Once deployed, point your SDK to your proxy URL instead of the default MGM endpoint:

**iOS**
```swift
MGM.configure(apiKey: "your-key", endpoint: "https://your-proxy.fly.dev")
```

**Android**
```kotlin
MostlyGoodMetrics.configure(context, "your-key", "https://your-proxy.fly.dev")
```

**Web**
```javascript
mgm.init({ apiKey: 'your-key', endpoint: 'https://your-proxy.fly.dev' });
```

**Server-side (Node.js example)**
```javascript
// When proxying from your server, make sure to forward the client IP!
await fetch('https://your-proxy.fly.dev/v1/events', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${apiKey}`,
    'Content-Type': 'application/json',
    'X-Forwarded-For': request.headers['x-forwarded-for'] || request.ip,
  },
  body: JSON.stringify({ events: [...] }),
});
```

## Health check

The proxy exposes a `/health` endpoint that returns `ok` with a 200 status. Use this for load balancer health checks.

```bash
curl https://your-proxy.fly.dev/health
# ok
```

## License

MIT — do whatever you want with it.
