---
name: vps-deploy
description: Deploy and operate a self-hosted service on a VPS over SSH (Docker, docker-compose), and run autonomous tasks that touch third-party accounts. Use when installing an app on a server, debugging a failed Docker install, or scheduling unattended jobs.
---

# VPS deploy — lessons learned

## Before touching the server
- Reuse existing access first: `~/.ssh/config`, existing keys, `ssh -o BatchMode=yes -o IdentitiesOnly=yes <host>`. Ask the user only if nothing works.
- Ambiguous product name (dictation, typo): confirm it on the official source before installing. Wrong product installed = say so, remove everything created (containers, volumes, dirs), restart clean.
- Compare the product's recommended hardware (RAM, CPU, disk) with the server's real resources (`free -h`, `nproc`, `df -h`) and warn on a gap before installing.

## Install and verify
- Run long builds in the background with `ServerAliveInterval` so the session survives; poll instead of blocking.
- `get.docker.com` fails on a duplicate apt source: `grep -r download.docker.com /etc/apt/sources.list /etc/apt/sources.list.d/`, remove the duplicate entry, rerun.
- Done = the real service answers (HTTP status code, container health), not "the command exited 0".
- Generated secrets stay on the server: never print them in chat, give the command to read them. Remind the user to back up the encryption key (losing it makes stored data unrecoverable).

## Unattended / scheduled tasks
- Guardrails read-only by default; writes limited to explicitly allowed directories.
- At start, check the real time and the account lock before interpreting anything; apply the fallback if a deadline has passed.
- Multi-account third-party tools: pass the account on every call, lock the task to the exact account id, and re-check it just before any send or publish.
