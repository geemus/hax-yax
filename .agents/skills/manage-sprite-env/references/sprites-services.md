# Sprite Internal Services & Checkpoints

Reference for `sprite-env` — the CLI available **from within a sprite instance** for managing local services, checkpoints, and environment info.

> **Do not use MCP Sprite tools** (e.g. `checkpoint_create`, `service_start`). They target remote sprites by name and will fail inside a local instance. Use `sprite-env` exclusively.

## Environment Info

```sh
sprite-env info           # Show instance ID, URL, org, active services
```

## Services

Long-running processes managed by the sprite runtime. Services persist across reboots and restart automatically.

```sh
# Create a service
sprite-env services create <name> \
  --cmd <binary-path> \
  --args "<arg1>,<arg2>" \
  --http-port 8080

# Only one service can own --http-port; it auto-starts on incoming HTTP requests.
# --cmd takes the binary only — pass arguments via --args (comma-separated):
#   WRONG:  --cmd "python3 -m http.server 8080"
#   RIGHT:  --cmd python3 --args "-m,http.server,8080" --http-port 8080

sprite-env services list
sprite-env services start <name>
sprite-env services stop <name>
sprite-env services restart <name>   # Preferred over stop + start
sprite-env services delete <name>
sprite-env services --help           # Full option reference
```

### Service logs

```sh
ls /.sprite/logs/services/           # Log files per service
cat /.sprite/logs/services/<name>    # Tail a specific service log
```

## Checkpoints

Point-in-time snapshots of the writable filesystem overlay. Only overlay data is captured (copy-on-write).

```sh
sprite-env checkpoints create                          # Snapshot current state
sprite-env checkpoints create --comment "description"  # With descriptive comment
sprite-env checkpoints list
```

Checkpoints are fast — create one whenever the instance reaches a known-good state. The last 5 checkpoints are mounted at `/.sprite/checkpoints/v<N>/`.

**Restore**: use the external `sprite restore <id>` command. Restore drops the entire current session — always confirm with the user before restoring.

## Network Policy

```sh
cat /.sprite/policy/network.json     # Allowed egress domains
```

Respect the domain allowlist. Avoid raw IP addresses unless resolved from an allowed domain.

## Context Files

| Path | Contents |
|------|----------|
| `/.sprite/llm.txt` | Platform behavior: services, checkpoints, filesystem, network |
| `/.sprite/docs/agent-context.md` | Full environment overview |
| `/.sprite/docs/languages.md` | Language runtimes and version managers |
| `/.sprite/languages/<lang>/llm.txt` | Per-language runtime details |
| `/.sprite/logs/services/` | Service log files |
| `/.sprite/checkpoints/v<N>/` | Last 5 checkpoint snapshots |
