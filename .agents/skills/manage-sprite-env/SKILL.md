---
name: manage-sprite-env
description: >
  Manages services, checkpoints, and environment info from within a running
  sprite instance using the `sprite-env` CLI.
  TRIGGER when: user is operating from within a sprite instance and references
  service management, in-sprite checkpoints, or environment inspection —
  including "start a service on this sprite", "create a service",
  "checkpoint this instance", "check my sprite environment", or "view network policy".
  DO NOT TRIGGER when: the user is managing sprites from outside (creating, destroying,
  executing commands remotely) — use manage-sprites instead.
license: Apache-2.0
metadata:
  author: geemus
  version: "1.0"
---

# Manage Sprite Env

Manages services, checkpoints, and environment info from within a running sprite instance using the `sprite-env` CLI.

For full `sprite-env` command reference, read `references/sprites-services.md`.

## Instructions

### 1. Confirm you are inside a sprite

Run `sprite-env info` to confirm the environment. If the command is not found or fails, you are likely outside a sprite — use the `manage-sprites` skill instead.

### 2. Read available context files

Before executing operations, read the following files if they exist:
- `/.sprite/llm.txt` — platform behavior overview
- `/.sprite/docs/agent-context.md` — full environment overview

These files inform service configuration and network policy decisions.

### 3. Execute the requested operation

**Environment info**
```sh
sprite-env info
```
Shows instance ID, URL, org, and active services.

**Services**
```sh
sprite-env services create <name> --cmd <binary> --args "<arg1>,<arg2>" --http-port 8080
sprite-env services list
sprite-env services start <name>
sprite-env services stop <name>
sprite-env services restart <name>   # preferred over stop + start
sprite-env services delete <name>    # destructive — confirm with user before running
```
Only one service can own `--http-port`. Pass arguments via `--args` (comma-separated), not inline in `--cmd`:
- Wrong: `--cmd "python3 -m http.server 8080"`
- Right: `--cmd python3 --args "-m,http.server,8080" --http-port 8080`

Before running `services delete`, state the service name to the user and ask for confirmation. Do not proceed until confirmed.

**Logs**
```sh
ls /.sprite/logs/services/
cat /.sprite/logs/services/<name>
```
Use to diagnose a failing or misbehaving service.

**Checkpoints**
```sh
sprite-env checkpoints create --comment "description"
sprite-env checkpoints list
```
Create a checkpoint whenever the instance reaches a known-good state. The last 5 checkpoints are available at `/.sprite/checkpoints/v<N>/`.

**Restore**: use the external `sprite restore <id>` command from outside the sprite. Always confirm with the user before restoring — it drops the current session state.

**Network policy**
```sh
cat /.sprite/policy/network.json
```
Shows allowed egress domains. Avoid raw IP addresses unless resolved from an allowed domain.

### 4. Surface results

Relay the raw command output to the user without modification. If the output indicates an error (non-zero exit, error message), explain what failed and suggest a corrective action. If `sprite-env` is not found, advise the user that this command is only available inside a sprite instance and suggest using `manage-sprites` for external operations.

## Examples

**Inspect environment:**
```sh
sprite-env info
```
Expected output:
```
instance: abc123
url: https://abc123.sprites.dev
org: my-org
services: web-server (running)
```

**Create a service:**
```sh
sprite-env services create web-server \
  --cmd python3 \
  --args "-m,http.server,8080" \
  --http-port 8080
```

**Service already exists (error):**
If `services create` returns `Error: service 'web-server' already exists`, run
`sprite-env services list` to confirm state, then use `services start <name>` if
the service exists but is stopped, or choose a different name.

**Checkpoint before a change:**
```sh
sprite-env checkpoints create --comment "before dependency upgrade"
```

**List checkpoints:**
```sh
sprite-env checkpoints list
```

**View service logs:**
```sh
ls /.sprite/logs/services/
cat /.sprite/logs/services/<name>
```
