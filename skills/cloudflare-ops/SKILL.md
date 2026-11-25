---
name: cloudflare-ops
description: Deploy a Worker to a custom domain, manage DNS/WAF/zone settings, and audit a Cloudflare zone's security. Use when publishing to Cloudflare, restricting access by IP, touching DNS records, or when the user asks "is my Cloudflare clean/secure".
---

> Install and prefer Cloudflare's official skills first (see `platform-official-skills`); this skill only adds cross-cutting operating rules.

# Cloudflare operations — lessons learned

## Pick the right tool
- **Wrangler** = Workers and their resources only (D1, KV, R2, secrets, routes/custom domains). Its OAuth token can NOT read/edit DNS or WAF.
- **`cf`** (official CLI, `npm i -g cf`, `cf auth login`, separate login from Wrangler) = the whole public API: zones, DNS, settings, rulesets. Discover commands with `cf cli search "<task>"`.
- If a `cf` subcommand's flags are unclear (nested settings values), call the REST API with `curl` and the `cf` OAuth token (stored in the CLI's local config directory). Never print the token.
- Before telling the user "the CLI can't do it", search for current tooling (new CLIs ship often). Saying "can't" from memory wastes a round trip.

## First Worker deploy on a fresh account
1. `wrangler login --browser=false` in background, grep the URL, give it to the user, poll `wrangler whoami`.
2. `wrangler d1 create <name>` → paste `database_id` into `wrangler.jsonc`, then `wrangler d1 migrations apply <name> --remote`.
3. Error 10063 "need a workers.dev subdomain": opening the dashboard does not always fix it. Create it via API: `PUT /accounts/:id/workers/subdomain {"subdomain":"<name>"}`.
4. Custom domain: `"routes": [{ "pattern": "app.example.com", "custom_domain": true }]`. Error 100117 "externally managed DNS records" → an old record exists. Show it to the user and get consent before deleting; deleting may also break other things (e.g. a dev tunnel hostname).
5. Generated secrets: write a JSON file in the scratchpad with `umask 077`, `wrangler secret bulk file.json`, delete the file. Third-party credentials: let the user type them with `wrangler secret put NAME` (hidden prompt). Never ask for secrets in chat.
6. Verify: `/` → 200, an unauthenticated API route → 401, an unsigned webhook → 400 (proves it reaches the app). A fresh workers.dev cert can fail TLS for a few minutes.

## Restrict an app to the owner's IP (free WAF custom rule)
- Expression, action Block:
  `(http.host eq "<app-hostname>" and not starts_with(http.request.uri.path, "<webhook-path-prefix>") and not ip.src in {<IPv4> <IPv6-prefix>::/64})`
- Always exclude inbound webhook paths (they stay protected by signature checks).
- Get BOTH addresses: WSL/servers may only see IPv4 while the user's browser uses IPv6 (many ISPs assign IPv6 by default). Ask the user to check their public address with a dual-stack IP echo service (e.g. https://api64.ipify.org). Allow the IPv6 /64, not a single address (privacy addresses rotate).
- Warn: dynamic IP / mobile data will be blocked; edit the rule to recover.

## Zone security audit checklist (read-only first, report, then fix with consent)
| Check | Good state |
|---|---|
| DMARC | exactly ONE `_dmarc` TXT (several = DMARC ignored) |
| Domain that sends no mail | `v=spf1 -all`, empty DKIM `*._domainkey "v=DKIM1; p="`, DMARC `p=reject` |
| Domain that receives no mail | null MX `0 .` instead of leftover MX |
| `min_tls_version` | 1.2 |
| HSTS (`security_header`) | enabled, moderate max-age (e.g. 6 months), no preload until sure |
| `ssl` | full (strict if origin cert valid) |
| DNSSEC | `active`; `pending` = DS record never added at the registrar (user action) |
| Origins | 52x on apex = dead origin → fix or delete records |
| CNAMEs to third-party hosts (Netlify, Vercel, etc.) | target still exists, else subdomain takeover risk |
| Leftovers | ftp records, old tunnels, unused API tokens |
| Bot Fight Mode | OFF on zones receiving platform webhooks (it challenges them) |
Fix the safe items (TLS, HSTS, duplicate DMARC, dead records) after one confirmation; DNSSEC at the registrar and site decisions belong to the user. Verify each change live (DNS-over-HTTPS `cloudflare-dns.com/dns-query` when `dig` is missing, `curl -I`, `curl --tls-max 1.1`).
