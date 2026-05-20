# OpenSourceHub

![Live](https://img.shields.io/badge/Live-opensourcehub.lovable.app-7b2fff?style=for-the-badge)
![Stack](https://img.shields.io/badge/Stack-React%20%2F%20Supabase%20%2F%20GraphQL-f0f0f0?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Live-3ecf8e?style=for-the-badge)

A GitHub repository discovery and health scoring tool for developers who want to find quality open source projects worth contributing to — without opening twenty tabs and guessing from star counts alone.

**Live:** [opensourcehub.lovable.app](https://opensourcehub.lovable.app)

---

## The Problem

Finding good open source projects to contribute to is harder than it should be. GitHub search returns results by stars or recency — neither tells you if a project is actively maintained, beginner-friendly, or worth your time.

OpenSourceHub scores repository health and surfaces the signal beneath the noise.

---

## What It Does

- **Health scoring** — rates repositories on activity, issue response time, contributor diversity, and maintenance signals
- **Beginner filters** — surfaces projects with good-first-issue density and welcoming contributor patterns
- **Trending detection** — identifies repos gaining momentum before they hit the front page
- **Zero-auth browsing** — no login required, no tracking, no friction

---

## Stack

| Layer | Tech | Why |
|---|---|---|
| Frontend | React + Vite + Tailwind | Fast dev, component-driven UI |
| Data layer | Supabase (PostgreSQL) | Structured cache storage + edge functions |
| API | GitHub GraphQL API | Precise field selection, lower rate limit cost than REST |
| Edge layer | Supabase Edge Functions | Token protection, response shaping before client sees data |
| Hosting | Lovable (Vite deploy) | Zero-config deployment |

---

## Architecture

```
Client (React)
    │
    ▼
Supabase Edge Function
    ├── Checks cache table (6-hour TTL)
    │       └── Cache hit → return shaped data to client
    │
    └── Cache miss → GitHub GraphQL API
            ├── GitHub token stored in Edge Function env (never client-side)
            ├── Fetch repo data with precise GraphQL query
            ├── Shape + score response
            ├── Write to Supabase cache table
            └── Return shaped data to client
```

The client never touches the GitHub API directly. All GitHub tokens live exclusively in Supabase Edge Function environment variables — they are never shipped to the browser.

---

## Key Technical Decisions

### GraphQL over REST
GitHub's REST API returns large payloads with fields the app doesn't need. GraphQL lets the edge function ask for exactly the fields required for scoring — fewer bytes, lower rate limit cost per request.

### 6-Hour Cache TTL
Repository health signals don't change minute to minute. A 6-hour cache window means repeated searches for the same repo return instantly from Supabase without burning GitHub API quota. Cache is invalidated on the next request after TTL expires.

### Edge Function as Proxy
Putting the GitHub token in a Supabase Edge Function rather than the frontend means:
- Token is never visible in browser network tabs
- Token is never in client-side bundle
- Rate limit handling and response shaping happen server-side before the client receives anything

### Supabase RLS
Row Level Security is enabled on all tables. The anon key (used client-side) can only read cached data. Writes happen exclusively through the service role key inside Edge Functions — never from the browser.

---

## What Was Hard

**Rate limiting without a backend.** GitHub's GraphQL API has a point-based rate limit system, not a simple request count. Each field in a query costs points. The edge function had to be designed to request the minimum viable field set while still gathering enough data to produce a meaningful health score.

**Mobile-only development.** This project was built and shipped entirely from an Android phone using Acode. No local dev server, no hot reload, no browser DevTools. Every architecture decision had to be right the first time because debugging in production is the only option.

**Scoring model calibration.** Defining what "healthy" means for a repository required iteration. A repo with 10,000 stars but no commits in 18 months scores lower than a 200-star repo with weekly releases and active issue responses.

---

## Repo Note

This repository contains architecture documentation only. The source code is managed privately due to environment configuration requirements with the deployment platform. The live application is publicly accessible at the link above.

---

## Author

**Rohit Kumar — DarkWing Studio**

| Platform | Link |
|---|---|
| Portfolio | [darkwingstudio.github.io](https://darkwingstudio.github.io) |
| GitHub | [github.com/DarkWingStudio](https://github.com/DarkWingStudio) |
| X | [x.com/darkwingstudio](https://x.com/darkwingstudio) |
