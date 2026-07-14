---
layout: single
title: "Self-hosted prerender service for SPA SEO"
date: 2026-07-14 10:24:38 -0600
categories: node typescript seo puppeteer fastify nginx docker
---

Single-page applications are great for building rich user experiences, but they can still be challenging for SEO bots, link previews, and crawlers. Some crawlers execute JavaScript well, some do not, and social preview bots usually expect useful HTML immediately.

I created a self-hosted prerender service to solve this problem for JavaScript-heavy SPAs. The project is available on GitHub: [https://github.com/haku-d/prerender](https://github.com/haku-d/prerender)

## What this project does

The service detects bot traffic and serves pre-rendered HTML instead of the normal JavaScript application shell. Regular users still receive the SPA as usual, while crawlers receive a fully rendered HTML snapshot.

Key features:

- Detects SEO bots and social crawlers from `User-Agent`
- Uses `Nginx` to route bot requests to the prerender service
- Renders pages with `Puppeteer Core` and headless Chromium
- Stores rendered HTML in a disk cache
- Uses `CACHE_TTL_SECONDS` to refresh stale pages
- Warms the cache by crawling the site's XML sitemap
- Supports `410 Gone` markers for removed pages
- Supports `301 Redirect` mappings for old URLs
- Stores bot analytics in `PostgreSQL`
- Exposes an analytics API for bot traffic and cache status

## How it works

At a high level, the architecture is simple:

1. `Nginx` receives every request.
2. Static assets such as JavaScript, CSS, images, and fonts are served normally.
3. For regular users, `Nginx` proxies the request to the SPA origin.
4. For known bots, `Nginx` forwards the request to the prerender service.
5. The prerender service checks whether a cached HTML file already exists.
6. If the cache is fresh, it returns the cached HTML immediately.
7. If the cache is missing or stale, Puppeteer opens the page, waits for hydration, extracts the HTML, minifies it, saves it to disk, and returns it.
8. Bot request details are optionally stored in PostgreSQL for analytics.

The important part is that regular users are not affected. Only bots and crawlers go through the rendering pipeline.

## Why self-host it?

There are existing hosted prerender services, but I wanted more control over the full pipeline:

- Control the bot detection rules
- Keep rendered HTML cache on my own infrastructure
- Avoid sending crawler traffic through a third-party SaaS service
- Add custom cache invalidation behavior
- Add analytics around bot traffic, cache hit rate, and render duration
- Integrate it with my own deployment and Nginx setup

For small and medium applications, this approach can be cost-effective because most bot requests are served from disk cache after the first render.

## Tech stack

| Layer | Technology |
|---|---|
| Runtime | Node.js 20, TypeScript |
| HTTP framework | Fastify 5 |
| Browser rendering | Puppeteer Core + Chromium |
| Cache storage | Disk cache |
| Database | PostgreSQL 16 |
| Migration tool | dbmate |
| Scheduler | toad-scheduler |
| HTML optimization | html-minifier-terser |
| Reverse proxy | Nginx |
| Deployment | Docker Compose |

## Docker services

The repository includes a `docker-compose.yml` with three services:

| Service | Purpose |
|---|---|
| `prerender` | Main Fastify service running Puppeteer and cache logic |
| `postgres` | Optional analytics database |
| `prerender-ui` | Optional dashboard for analytics and management |

The minimum environment configuration looks like this:

```env
PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium-browser
SITE_URL=https://your-site.com
STATIC_SITE_DIR=/data/prerender
DATABASE_URL=postgresql://prerender:secret@postgres:5432/prerender
POSTGRES_PASSWORD=secret
CACHE_TTL_SECONDS=43200
EXECUTE_JOB_ON_START=false
```

Then start the service:

```bash
docker compose up -d
```

If analytics are not required, `postgres`, `DATABASE_URL`, and `prerender-ui` can be removed.

## Nginx bot detection

The key part is the `Nginx` configuration. It maps bot user agents to a prerender flag:

```nginx
map $http_user_agent $prerender_ua {
    default 0;
    "~*googlebot" 1;
    "~*bingbot" 1;
    "~*facebookexternalhit" 1;
    "~*twitterbot" 1;
    "~*linkedinbot" 1;
    "~*slackbot" 1;
    "~*discordbot" 1;
}
```

Static assets are excluded so the service only renders real pages:

```nginx
map $uri $prerender {
    default $x_prerender;
    "~*\.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)" 0;
}
```

When `$prerender` is enabled, the request is rewritten to the prerender backend:

```nginx
location / {
    if ($prerender = 1) {
        rewrite (.*) /prerender-proxy last;
    }

    proxy_pass http://s3_bucket;
}

location /prerender-proxy {
    rewrite .* /https://$host$request_uri? break;
    proxy_set_header X-Prerender 1;
    proxy_pass http://prerender_backend;
}
```

The `X-Prerender` header is important because it prevents a request from being routed back into the prerender flow again.

## Cache behavior

Rendered pages are stored as HTML files under `STATIC_SITE_DIR`. Each cache file is keyed by an MD5 hash of the full canonical URL.

The service supports these cache states:

| Status | Meaning |
|---|---|
| `hit` | Fresh cached HTML was returned |
| `miss` | No cached file existed, so Puppeteer rendered the page |
| `stale` | Cached HTML existed but passed the TTL and needed refresh |
| `gone` | URL was marked as permanently removed |
| `redirect` | URL matched a stored redirect mapping |

The default TTL is `43200` seconds, or 12 hours.

## Useful endpoints

The service is intended to run privately behind `Nginx`, so the API is intentionally simple.

### Force refresh one page

```http
GET /hit/https://example.com/products/business-cards
```

This re-renders the page and overwrites the cache entry.

### Mark a page as gone

```http
GET /gone/https://example.com/old-page
```

Future bot requests for that URL return `410 Gone`.

### Add a redirect

```http
GET /redirect/https://example.com/old-path/https://example.com/new-path
```

Future bot requests for the old URL return a `301` redirect.

### View analytics summary

```http
GET /analytics/summary?from=2026-07-01&to=2026-07-14
```

Example response:

```json
{
  "total": 1240,
  "uniquePages": 87,
  "byBot": [{ "botName": "googlebot", "count": 640 }],
  "byCacheStatus": [{ "cacheStatus": "hit", "count": 980 }]
}
```

## What I learned

The biggest lesson is that prerendering is not only about running Puppeteer. The browser rendering part is important, but the real value comes from the surrounding system:

- Good bot detection
- Avoiding static asset rendering
- Preventing proxy loops
- Cache freshness and invalidation
- Sitemap-based cache warming
- Handling deleted and redirected pages correctly
- Measuring cache hit rate and render performance

A prerender service becomes much more useful when it behaves like infrastructure, not just a script that opens pages in Chromium.

## Source code

You can find the full project here:

[https://github.com/haku-d/prerender](https://github.com/haku-d/prerender)
