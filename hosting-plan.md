# E-Commerce SaaS Hosting Plan (Year 1 — 50 Clients)

## Approach

- Single application, subdomain-per-store
- One codebase routing to individual stores
- Keep it simple — don't over-engineer at 50 clients

## Minimum Infrastructure

| Resource                       | Spec                                  | Est. Cost/Month                     |
| ------------------------------ | ------------------------------------- | ----------------------------------- |
| VPS (app + DB)                 | 4 vCPU / 8 GB RAM                     | ~$48 (Hetzner) – $96 (DigitalOcean) |
| Managed Database               | Optional at 50 users; run on same VPS | $0                                  |
| Object Storage (images/assets) | Wasabi or Backblaze B2                | ~$6–10                              |
| CDN                            | Cloudflare (free tier)                | $0                                  |
| Email Delivery                 | Mailgun Flex or Postmark              | ~$15                                |
| Backups                        | VPS snapshots or Borg to B2           | ~$5                                 |
| Domain                         | 1 domain                              | ~$1                                 |
| **Total**                      |                                       | **~$75–120/month**                  |

## What You'll Need to Build

- **Tenant routing** — subdomain (`store.yourdomain.com`) or custom domain per store
- **Automated onboarding** — form to create store, database, subdomain
- **SSL automation** — Caddy or acme.sh with Let's Encrypt
- **Billing/invoicing** — Stripe integration
- **Admin panel** — manage stores, monitor usage

## Suggested Stack

- [Medusa.js](https://medusajs.com) — open-source headless commerce, multi-store capable
- PostgreSQL
- Redis (sessions, caching)
- Caddy as reverse proxy (auto-SSL)
- Cloudflare for CDN/DNS

## Visitor Capacity (Per Store)

Based on 4 vCPU / 8 GB RAM across 50 stores (~100–200 req/s total, ~2–4 req/s per store):

| Metric              | Light Store | Average Store | Busy Store |
| ------------------- | ----------- | ------------- | ---------- |
| Concurrent visitors | 10–20       | 5–10          | 2–5        |
| Page loads/hour     | 500–1000    | 200–500       | 50–200     |
| Daily visitors      | 1000–2000   | 500–1000      | 100–500    |

- A few busy stores can spike to **2000+ daily visitors** if others are idle
- **Bottleneck**: product images — offload to CDN (Cloudflare free tier handles this)
- Realistically, most stores will be low-traffic early on, letting the server handle a handful of moderate-traffic stores comfortably

## Scaling Triggers (beyond Year 1)

| Condition             | Action                                           |
| --------------------- | ------------------------------------------------ |
| 100+ clients          | Add a second app server + load balancer          |
| DB becomes bottleneck | Migrate to managed database (RDS, DO Managed DB) |
| High traffic stores   | Isolate them on dedicated resources              |
| Growing storage needs | Increase object storage tier                     |

---

_This is a realistic minimum for year one. Revenue from 50 stores should far outweigh hosting costs, making it a healthy margin._
