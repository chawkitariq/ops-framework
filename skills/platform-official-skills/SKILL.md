---
name: platform-official-skills
description: Avant de travailler avec Cloudflare ou Vercel (déploiement, config, Workers, Functions, DNS, sécurité, perf), vérifier que les skills officiels de la plateforme sont installés et s'appuyer dessus en priorité. Inclut l'audit de sécurité Cloudflare.
---

# Skills officiels des plateformes (Cloudflare, Vercel)

Les éditeurs maintiennent leurs propres skills, à jour avec leurs produits. Ils font foi sur le fonctionnement de la plateforme ; ce framework ne les recopie pas, il impose de les utiliser.

## Règle

Dès qu'une tâche touche Cloudflare ou Vercel :

1. **Vérifier** que les skills officiels de la plateforme sont disponibles (liste des skills / plugins de la session).
2. **Sinon, proposer de les installer** (demander l'accord avant toute installation) avec les commandes ci-dessous.
3. **Les charger et les suivre en priorité** pour tout ce qui est propre à la plateforme (API, CLI, limites, configuration, bonnes pratiques). Ce framework ne couvre que les règles transverses (architecture, sécurité générale, méthode).
4. En cas de contradiction avec un skill générique, **le skill officiel, plus récent, l'emporte** sur les faits de la plateforme ; vérifier la doc officielle en cas de doute.

## Cloudflare

- Dépôt : https://github.com/cloudflare/skills (≈25 skills : `cloudflare` pour s'orienter, `wrangler`, `cf-cli`, `workers-best-practices`, `frameworks-on-cloudflare`, `durable-objects`, `agents-sdk`, `turnstile-spin`, `web-perf`, Zero Trust…) + le serveur MCP Cloudflare.
- Installation Claude Code :
  ```
  /plugin marketplace add cloudflare/skills
  /plugin install cloudflare@cloudflare
  ```
  Alternative : `npx skills add https://github.com/cloudflare/skills`
- Point d'entrée : le skill `cloudflare`, qui oriente vers le bon produit et le bon skill.

### Audit de sécurité

- Dépôt : https://github.com/cloudflare/security-audit-skill (skill `security-audit`).
- Installation : `npx skills add https://github.com/cloudflare/security-audit-skill --skill security-audit` (`--global` pour tous les projets).
- À utiliser pour tout audit de sécurité d'un code (avant mise en production, après un changement sensible d'auth, de paiement ou d'exposition publique), quelle que soit la plateforme. Il exige un bac à sable de l'OS pour exécuter le code cible ; sans bac à sable, les résultats restent « à valider ».

## Vercel

- Annuaire : https://vercel.com/docs/agent-resources/skills (déploiement `deploy-to-vercel`, `vercel-cli`, React/Next.js `vercel-react-best-practices`, AI SDK `ai-sdk`, `workflow`, `sandbox`…).
- Installation du plugin Vercel (skills + agents + MCP) : `npx plugins add vercel/vercel-plugin`
- Ou skill par skill : `npx skills add <owner/repo> --skill <nom>` (ex. `npx skills add vercel-labs/agent-skills --skill deploy-to-vercel`). Recherche : `npx skills find <requête>`.
- Le MCP Vercel demande une authentification (`/mcp`) avant usage.

## Done when

- Les skills officiels de la plateforme concernée sont installés (ou leur installation a été proposée et refusée).
- Les décisions propres à la plateforme s'appuient sur eux ou sur la doc officielle, pas sur des souvenirs.
