# OPERATOR-QUICKSTART.md

Fast, no-drama operating guide for Kite + Drift.

## A) 60-Second Triage (always first)
Run on the host you are touching:

```bash
hostname
openclaw gateway probe
openclaw status --deep | sed -n '/^Channels/,$p'
```

If host/channel does not match intent, stop and re-anchor before changing anything.

---

## B) Gateway Restart SOP (safe order)
Use this when gateway is flaky, channels stop responding, or after config changes.

### 1) Confirm host and role
- Pi (`donkey`) owns Telegram/Kite
- Droplet owns Discord/Drift

### 2) Install/sync service definition (prevents token drift)
```bash
openclaw gateway install --force
systemctl --user daemon-reload || true
```

### 3) Restart gateway
```bash
openclaw gateway restart
sleep 2
```

### 4) Verify health
```bash
openclaw gateway probe
openclaw status --deep | sed -n '/^Channels/,$p'
```

### 5) If RPC timeout persists
```bash
lsof -i :18789
openclaw logs --follow
```

---

## C) Restore Dashboard Access (local)
OpenClaw dashboard is local to the host where gateway runs.

### If you are on the same host
Open:
- `http://127.0.0.1:18789`

### If accessing Droplet dashboard from your PC
Use SSH tunnel (keep terminal open):

```bash
ssh -L 18791:127.0.0.1:18789 russ@134.122.12.212
```
Then browse:
- `http://127.0.0.1:18791`

### If accessing Pi dashboard from your PC
Use different local port:

```bash
ssh -L 18790:127.0.0.1:18789 russ@<PI_IP>
```
Then browse:
- `http://127.0.0.1:18790`

### If tunnel fails with "Address already in use"
Pick another local port (e.g., 18792):

```bash
ssh -L 18792:127.0.0.1:18789 russ@<HOST>
```

---

## D) Hard Guardrails (Scope Lock)

## Drift (Droplet) scope
- Discord operations only
- Loop/worker workloads
- Runtime triage on droplet
- Must keep Telegram OFF

## Kite (Pi) scope
- Telegram operations only
- Canonical workspace/docs/truth source
- Final policy/runtime authority
- Must keep Discord OFF

### Forbidden
- Same channel ON on both nodes
- Blind full `.openclaw` sync between nodes
- Architecture changes during active incident response

---

## E) Incident Rule: One Change Class at a Time
1. Identify host (`hostname`)
2. Isolate one variable (channel, token, service, version)
3. Apply one fix
4. Verify
5. Only then move to next fix

---

## F) Stabilization Verification Matrix (must pass)

### Pi (Kite)
- `hostname` => donkey
- Telegram ON
- Discord OFF
- `openclaw gateway probe` => Reachable yes + RPC ok

### Droplet (Drift)
- `hostname` => ubuntu-s-1vcpu-2gb-nyc3-01
- Discord ON
- Telegram OFF
- `openclaw gateway probe` => Reachable yes + RPC ok

If any item fails: stop, revert to section B.
