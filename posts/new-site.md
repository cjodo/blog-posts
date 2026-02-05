# How I built this site

This post walks through how this site is built using **Astro**, deployed on **Railway**, with a **Cloudflare Worker** that fetches blog content from a **GitHub repository**.

---

## Overview

**Live site:** cjodo.com
**Source code:**  [Github](https://github.com/cjodo/astro-portfolio) 
**Deployment:** Railway  

## Cloudflare Worker for Blog Posts
```typescript
const GH_BASE =
    'https://raw.githubusercontent.com/cjodo/blog-posts/main'

export default {
    async fetch(req) {
        const url = new URL(req.url)

        // GET /post/posts/hello-world.md
        if (url.pathname.startsWith('/post/')) {
            const path = url.pathname.replace('/post/', '')

            return fetch(`${GH_BASE}/${path}`, {
                cf: { cacheTtl: 3600 } // 1 hour
            })
        }

        return new Response('Not found', { status: 404 })
    }
}


```

The goal of this site was to build a fast, mostly-static website with dynamic content pulled from GitHub at request time — without running a traditional CMS.

---

## Tech Stack

### Frontend
- **Framework:** Astro
- **Styling:** Tailwind 

### Content Fetching
- **Edge runtime:** Cloudflare Workers
- **Source:** GitHub repository (markdown files)

### Hosting & Infrastructure
- **Frontend hosting:** Railway
- **Worker hosting:** Cloudflare
- **Source control:** GitHub

---

## Project Goals

Primary goals:
- Keep the frontend light and mostly static
- Store content in GitHub
- Leverage edge caching for content reads
- Keep the architecture simple and cheap

---

## Architecture Overview

High-level Blog flow:

```text
User
↓
Astro Static Site (Railway)
↓
Cloudflare Worker API
↓
GitHub Content Repository
```
