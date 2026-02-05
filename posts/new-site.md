# How I build this site

This post walks through how this site is built using **Astro**, deployed on **Railway**, with a **Cloudflare Worker** that fetches blog content from a **GitHub repository**.

---

## Overview

**Live site:**  
**Source code:**  
**Content repo:**  
**Deployment:** Railway  
**Edge services:** Cloudflare Workers  

The goal of this site was to build a fast, mostly-static website with dynamic content pulled from GitHub at request time — without running a traditional CMS.

---

## Tech Stack

### Frontend
- **Framework:** Astro
- **Rendering mode:** Static site generation (SSG)
- **Styling:** 
- **Components:** Astro components with optional framework islands

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
- Keep the frontend fully static
- Store content in GitHub
- Avoid a traditional CMS
- Leverage edge caching for content reads
- Keep the architecture simple and cheap

Non-goals:
- Real-time content editing
- Heavy backend logic
- Complex auth flows

---

## Architecture Overview

High-level flow:

```text
User
  ↓
Astro Static Site (Railway)
  ↓
Cloudflare Worker API
  ↓
GitHub Content Repository

