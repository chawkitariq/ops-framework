# Ops Framework

Reusable [Agent Skills](https://code.claude.com/docs/en/skills) for hosting and infrastructure operations, distilled from real sessions with Claude Code. Part of the [framework](https://github.com/chawkitariq/framework) family, next to [dev-framework](https://github.com/chawkitariq/dev-framework) and [uiux-framework](https://github.com/chawkitariq/uiux-framework).

## Skills

| Skill | Use it when |
|---|---|
| [`cloudflare-ops`](skills/cloudflare-ops/SKILL.md) | Deploying a Worker to a custom domain, managing DNS/WAF with `cf`, restricting access by IP, auditing a zone's security |
| [`free-hosting-architecture`](skills/free-hosting-architecture/SKILL.md) | Choosing free or cheap hosting, or re-architecting an app for serverless |

## Agents

None yet (`agents/` is reserved).

## Install (Claude Code)

```bash
git clone https://github.com/chawkitariq/ops-framework.git
cp -r ops-framework/skills/* ~/.claude/skills/
cp ops-framework/agents/*.md ~/.claude/agents/ 2>/dev/null || true
```

## License

MIT
