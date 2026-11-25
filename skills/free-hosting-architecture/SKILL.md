---
name: free-hosting-architecture
description: Decide where and how to host an app for free or cheaply (Vercel, Cloudflare, Render, Fly.io, Railway, Oracle Free, VPS, local), and how to re-architect for serverless. Use when the user asks about hosting cost, free plans, "can it run on Vercel/Cloudflare", or a lighter version of an app.
---

# Free / cheap hosting — decision guide

## Step 1: find the blockers before talking prices
List every **long-lived connection** and background job: outbound WebSocket clients to third-party APIs, gRPC server streams (some APIs expose streams only over gRPC, with no REST equivalent), SSE to browsers, in-memory state (Redis/Valkey), cron.
Serverless functions (Vercel, Netlify, Lambda) cap duration → these break. Say so first; price is secondary.

## Step 2: options, honestly ranked
1. **Local machine** if the app is only used while the user's PC is on (e.g. a personal dashboard): free, full features, no rewrite. A tunnel (Cloudflare Tunnel) covers inbound webhooks. Often the true best answer — say it.
2. **Cheap VPS** (~4-7 €/month) runs the existing docker-compose unchanged.
3. **Oracle Always Free**: unchanged stack, free, but fragile: ARM allowances have been reduced over time, "out of host capacity" errors (a less busy home region helps), idle reclaim when CPU/net/mem p95 < 20 % for 7 days (bad for apps idle between uses; Pay-As-You-Go + budget alert mitigates).
4. **PaaS** (Railway ~5-10 $/mo, Fly ~6-10 $, Render ~13-20 $): always-on, simple; free tiers sleep or expire, so unusable for always-on.
5. **Rewrite to event-driven serverless**: free but a real project.

## Step 3: if rewriting for serverless, compare platform limits (example: Cloudflare vs Vercel)
- Example, Cloudflare free tier (verify current limits): Workers (daily request cap, very small CPU time per invocation), a SQLite-backed Durable Object for realtime (WebSocket/SSE fan-out + ring buffer), D1, cron triggers, static assets. Check each plan's terms for "non-commercial" clauses if the project is monetized (e.g. Vercel Hobby has one).
- Replace persistent sockets with **webhooks** when the upstream API offers a webhook transport. For stream-only (gRPC) APIs: poll a list endpoint driven by scheduled alarms only while needed (mind quota cost), or use a relay (local PC, a container platform).
- Check framework compatibility with the runtime: heavy Node frameworks/ORMs may not run on Workers (lighter alternatives such as Hono + Drizzle do; verify auth libraries too).
- **Single-tenant simplifies massively**: no tenancy, one stateful object, owner-only sign-up, tiny schema. Ask early whether multi-tenant is really needed.
- Keep the existing front and expose the same HTTP contract so the UI needs only a configuration change.

## Communication rules
- Give price ranges, label them unverified unless fetched today, cite sources when searched.
- When the user asks for "honest", give one recommendation and the trade-offs, not a menu.
- Validate risky assumptions with a small prototype before a full rewrite.
