---
name: free-hosting-architecture
description: Decide where and how to host an app for free or cheaply (Vercel, Cloudflare, Render, Fly.io, Railway, Oracle Free, VPS, local), and how to re-architect for serverless. Use when the user asks about hosting cost, free plans, "can it run on Vercel/Cloudflare", or a lighter version of an app.
---

# Free / cheap hosting — decision guide

## Step 1: find the blockers before talking prices
List every **long-lived connection** and background job: WebSocket clients (e.g. Twitch EventSub WS), gRPC server streams (e.g. YouTube `liveChatMessages.streamList` is gRPC-only, no REST stream), SSE to browsers, in-memory state (Redis/Valkey), cron.
Serverless functions (Vercel, Netlify, Lambda) cap duration → these break. Say so first; price is secondary.

## Step 2: options, honestly ranked
1. **Local machine** if the app is only used while the user's PC is on (e.g. a live-stream dashboard): free, full features, no rewrite. A tunnel (Cloudflare Tunnel) covers inbound webhooks. Often the true best answer — say it.
2. **Cheap VPS** (~4-7 €/month) runs the existing docker-compose unchanged.
3. **Oracle Always Free**: unchanged stack, free, but fragile: ARM A1 cut to 2 OCPU/12 GB (2026), "out of host capacity" (pick a less busy home region, e.g. Marseille not Paris), idle reclaim when CPU/net/mem p95 < 20 % for 7 days (bad for apps idle between uses; Pay-As-You-Go + budget alert mitigates).
4. **PaaS** (Railway ~5-10 $/mo, Fly ~6-10 $, Render ~13-20 $): always-on, simple; free tiers sleep or expire, so unusable for always-on.
5. **Rewrite to event-driven serverless**: free but a real project.

## Step 3: if rewriting, prefer Cloudflare over Vercel
- Cloudflare free: Workers (100k req/day, **10 ms CPU per invocation**), one SQLite Durable Object for realtime (WebSocket/SSE fan-out + ring buffer), D1, cron triggers, static assets, no "non-commercial" clause (Vercel Hobby has one — grey zone for a monetized creator).
- Replace persistent sockets with **webhooks** (Twitch EventSub supports webhook transport for chat). gRPC-only APIs: poll with `list` driven by DO alarms only while needed (quota cost!), or a relay (local PC, Cloud Run).
- NestJS/TypeORM don't run on Workers → Hono + Drizzle; Better Auth works on D1.
- **Single-tenant simplifies massively**: no tenancy, one DO, owner-only sign-up hook, tiny schema. Ask early whether multi-tenant is really needed.
- Keep the existing front and expose the same HTTP contract so the UI needs only a build flag.

## Communication rules
- Give price ranges, label them unverified unless fetched today, cite sources when searched.
- When the user asks for "honest", give one recommendation and the trade-offs, not a menu.
- Validate risky assumptions with a small prototype before a full rewrite.
