# Soulbound OpenClaw

Kite + Drift operating system for a two-node OpenClaw deployment.

## Architecture (Canonical)
- **Pi (`donkey`)**: Canonical control surface, canonical workspace/docs, Kite runtime.
- **DigitalOcean Droplet**: Worker node, Discord operations, loop workloads, Drift runtime.

## Channel Ownership (Hard Rule)
- **Pi / Kite**: Telegram **ON**, Discord **OFF**
- **Droplet / Drift**: Discord **ON**, Telegram **OFF**

Never enable the same channel on both nodes at once.

## Why this repo exists
This repository captures runtime policy, operating procedures, and incident learnings so the system remains stable under pressure and does not drift during recovery work.

## Current Reliability Lessons (Feb 26, 2026)
The outage/confusion pattern came from multiple variables changing at once:
1. Channel overlap across nodes (Telegram active on wrong node)
2. Service/token mismatch after edits/restarts
3. Version drift and restart timing issues
4. Mixing architecture changes while doing live incident recovery

### Preventive posture
- Fix one class of issue at a time.
- Confirm host identity (`hostname`) before applying changes.
- Apply channel ownership first, then restart, then verify.
- Keep Pi as source of truth; treat droplet outputs as worker artifacts.

## Fast Verification Commands
Run on each host:

```bash
hostname
openclaw gateway probe
openclaw status --deep | sed -n '/^Channels/,$p'
```

Expected steady state:
- Pi shows Telegram ON only
- Droplet shows Discord ON only
- Probe shows Reachable yes + RPC ok

## Core docs
- `SOUL-OBJECTIVES.md` — mission, constraints, guardrails
- `KITE-RUNBOOK.md` — operational SOPs, recovery playbooks

## Contribution flow
1. Make changes on the correct node role
2. Validate with probe + deep status
3. Commit with explicit incident/context message
4. Push to `main`
