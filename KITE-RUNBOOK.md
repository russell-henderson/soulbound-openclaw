# KITE-RUNBOOK.md

Operational procedures for Kite (Pi) and Drift (Droplet).

## 1) Role and Source-of-Truth
- **Canonical host:** Pi (`donkey`)
- **Worker host:** DigitalOcean droplet
- **Canonical docs/config decisions live on Pi** unless explicitly promoted.

## 2) Channel Separation SOP (Mandatory)
### Target state
- Pi: Telegram ON, Discord OFF
- Droplet: Discord ON, Telegram OFF

### Enforcement sequence
1. Disable Telegram on Droplet first.
2. Confirm Droplet channels.
3. Enable Telegram on Pi.
4. Restart Pi gateway.
5. Re-verify both nodes.

### Verify commands
```bash
hostname
openclaw status --deep | sed -n '/^Channels/,$p'
openclaw gateway probe
```

## 3) Incident Pattern: RPC Timeout While Service Appears Running
### Symptom
- `Connect: ok` but `RPC: failed - timeout`

### Common causes
- Token/config drift between CLI and service
- Restart collision with stale process on port 18789
- Mixed changes across nodes during recovery

### Recovery checklist
1. Confirm host identity (`hostname`).
2. Check live listener:
   ```bash
   lsof -i :18789
   ```
3. Sync service config:
   ```bash
   openclaw gateway install --force
   systemctl --user daemon-reload
   ```
4. Restart gateway:
   ```bash
   openclaw gateway restart
   ```
5. Verify:
   ```bash
   openclaw gateway probe
   openclaw status --deep
   ```

## 4) Token/Service Drift Guard
Run after major changes or suspicious restarts:

```bash
openclaw gateway install --force
openclaw gateway restart
openclaw gateway probe
```

## 5) Safety Rails During Recovery
- Do not change architecture and incident state at the same time.
- Do not paste command output back into shell.
- Do not assume host context; always run `hostname` first.
- Do not enable a channel on both nodes.

## 6) Dashboard Recovery (when UI seems unreachable)
- Dashboard is always local to the gateway host (`127.0.0.1:18789`).
- Use SSH tunnels from your PC:
  - Pi: `ssh -L 18790:127.0.0.1:18789 russ@<PI_IP>` → open `http://127.0.0.1:18790`
  - Droplet: `ssh -L 18791:127.0.0.1:18789 russ@134.122.12.212` → open `http://127.0.0.1:18791`
- If "address already in use", choose a different local port.

## 7) Scope Guardrails (enforced)
### Kite (Pi)
- Telegram ON
- Discord OFF
- Canonical docs and policy authority

### Drift (Droplet)
- Discord ON
- Telegram OFF
- Worker/loop execution lane only

### Guardrail checks
```bash
hostname
openclaw status --deep | sed -n '/^Channels/,$p'
openclaw gateway probe
```

## 8) Weekly Stability Routine
On both hosts:

```bash
openclaw gateway probe
openclaw status --deep
openclaw security audit --deep
openclaw update status
```

Record findings and action items in this repo.

## 9) Feb 26, 2026 Incident Summary
### What happened
- Multi-node context confusion and channel overlap during active recovery.

### What fixed it
- Strict channel ownership split
- Service/token sync
- Controlled restart and verification order

### Durable policy update
- **One channel, one owner node.**
- **One source of truth (Pi).**
- **One change class at a time.**
