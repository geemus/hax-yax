---
name: manage-sprite-env
description: >
  Manages services, checkpoints, and environment info from within a running
  sprite instance using the `sprite-env` CLI. Use when the user is inside a
  sprite and wants to create or manage services, take a checkpoint, inspect
  environment info, or check network policy.
  Also triggered by "start a service on this sprite", "create a service",
  "checkpoint this instance", "check my sprite environment", or "view network policy".
  TRIGGER when: user is operating from within a sprite instance and references
  service management, in-sprite checkpoints, or environment inspection.
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

### 2. Execute the requested operation

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
sprite-env services delete <name>
```
Only one service can own `--http-port`. Pass arguments via `--args` (comma-separated), not inline in `--cmd`:
- Wrong: `--cmd "python3 -m http.server 8080"`
- Right: `--cmd python3 --args "-m,http.server,8080" --http-port 8080`

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

### 3. Surface results

Return command output directly. If `sprite-env` is not found, advise the user that this command is only available inside a sprite instance and suggest using `manage-sprites` for external operations.

## Examples

**Inspect environment:**
```sh
sprite-env info
```

**Create a service:**
```sh
sprite-env services create web-server \
  --cmd python3 \
  --args "-m,http.server,8080" \
  --http-port 8080
```

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
