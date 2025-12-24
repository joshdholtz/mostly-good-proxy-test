# MGM Proxy

A lightweight reverse proxy for [Mostly Good Metrics](https://mostlygoodmetrics.com) that preserves client IP addresses for accurate geolocation.

[![Deploy to Fly.io](https://img.shields.io/badge/Deploy%20to-Fly.io-8b5cf6?style=for-the-badge&logo=fly.io)](https://fly.io/docs/speedrun/)
[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/template?code=mgm-proxy&referralCode=mostlygoodmetrics)
[![Deploy to Heroku](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/mostlygoodmetrics/mgm-proxy)
[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/mostlygoodmetrics/mgm-proxy)
[![Deploy to DO](https://www.deploytodo.com/do-btn-blue.svg)](https://cloud.digitalocean.com/apps/new?repo=https://github.com/mostlygoodmetrics/mgm-proxy/tree/main)

## Why use a proxy?

When your app sends analytics events through a CDN, load balancer, or serverless function, the original client IP gets replaced with the proxy's IP. This proxy extracts the real client IP from standard headers and forwards it to MGM.

## Features

- Zero dependencies (Go standard library only)
- ~5MB Docker image
- ~10MB memory usage
- Extracts client IP from: `CF-Connecting-IP`, `True-Client-IP`, `X-Real-IP`, `X-Forwarded-For`
- Health check endpoint at `/health`

## Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `MGM_TARGET_URL` | `https://ingest.mostlygoodmetrics.com` | MGM ingestion endpoint |
| `PORT` | `8080` | Port to listen on |

## Deploy to Fly.io

```bash
# Install flyctl if needed
curl -L https://fly.io/install.sh | sh

# Launch (first time)
cd mgm-proxy
fly launch --no-deploy
fly deploy

# Or deploy to a specific region
fly deploy --region iad
```

## Deploy to Heroku

```bash
# Using Docker (recommended)
heroku create my-mgm-proxy
heroku stack:set container
git push heroku main

# Or using Go buildpack
heroku create my-mgm-proxy
heroku buildpacks:set heroku/go
git push heroku main
```

## Deploy to Digital Ocean App Platform

1. Fork this repo or push to your own GitHub
2. Go to [Digital Ocean App Platform](https://cloud.digitalocean.com/apps)
3. Create App → Select your repo
4. It will auto-detect the Dockerfile
5. Set environment variable `MGM_TARGET_URL` if needed
6. Deploy

## Deploy to Railway

```bash
# Install Railway CLI
npm install -g @railway/cli

# Deploy
railway login
railway init
railway up
```

## Deploy to Render

1. Go to [Render Dashboard](https://dashboard.render.com)
2. New → Web Service
3. Connect your repo
4. Render will auto-detect the Dockerfile
5. Deploy

## Run with Docker

```bash
docker build -t mgm-proxy .
docker run -p 8080:8080 -e MGM_TARGET_URL=https://ingest.mostlygoodmetrics.com mgm-proxy
```

## Run locally

```bash
go run main.go
```

## Usage

Point your SDK to your proxy URL instead of the MGM ingestion endpoint:

```swift
// iOS
MGM.configure(apiKey: "your-key", endpoint: "https://your-proxy.fly.dev")
```

```kotlin
// Android
MostlyGoodMetrics.configure(context, "your-key", "https://your-proxy.fly.dev")
```

```javascript
// Web
mgm.init({ apiKey: 'your-key', endpoint: 'https://your-proxy.fly.dev' });
```

## License

MIT
