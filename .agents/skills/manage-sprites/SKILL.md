---
name: manage-sprites
description: >
  Provisions, operates, and maintains sprites.dev instances through the full
  lifecycle: authenticate, create, execute commands, manage checkpoints, and
  destroy. Use when the user asks to create a sprite, run a command on a sprite,
  checkpoint or restore a sprite, open a shell on a sprite, or delete a sprite.
  Also triggered by "spin up a sprite", "exec on my sprite", "snapshot my sprite",
  "roll back my sprite", "tear down a sprite", "get my sprite URL", "make my sprite
  public", or "proxy a port on my sprite".
  TRIGGER when: user references sprite lifecycle operations, sprites.dev, or managing
  remote sandbox instances.
  DO NOT TRIGGER when: the user is operating from within a sprite and managing local
  services or checkpoints — use manage-sprite-env instead.
license: Apache-2.0
metadata:
  author: geemus
  version: "1.0"
---

# Manage Sprites

Manages the full lifecycle of sprites.dev instances using the `sprite` CLI.

For CLI command details, read `references/sprites-cli.md`.

## Instructions

### 1. Confirm authentication

Run `sprite list` to confirm auth is working. If it fails, run `sprite login` (browser flow) or `sprite auth setup --token <token>` (token flow) and wait for the user to complete it.

### 2. Identify the target sprite

- If the user specifies a sprite name or has run `sprite use`, proceed with that sprite.
- If ambiguous, run `sprite list` and ask the user to confirm.
- For commands that accept `-s <name>`, prefer the explicit flag over relying on an active sprite.

### 3. Execute the requested lifecycle operation

Follow the minimal path for the operation:

**Create**
```
sprite create <name>
```
Optionally scope to an org: `sprite create -o <org> <name>`.
After creation, run `sprite use <name>` if the user wants it set as the active sprite for the directory.

**Execute a command**
```
sprite exec -s <name> <command> [args...]
```
For an interactive shell: `sprite console -s <name>`.

**Checkpoint**
```
sprite checkpoint create -s <name>
sprite checkpoint list -s <name>
```
Recommend creating a checkpoint before any operation that modifies installed software, configuration files, or running services — e.g., `apt install`, editing config files, or service restarts.

**Restore**
```
sprite restore <checkpoint-id>
```
Always confirm with the user before restoring — it drops the current session state.

**Destroy**
```
sprite destroy <name>
```
Always confirm with the user before destroying — this is irreversible.

**URL and Proxy**
```
sprite url                            # Show public URL for active sprite
sprite url update --auth public       # Make URL publicly accessible
sprite url update --auth sprite       # Restrict to org members (default)
sprite proxy <port> [port2...]        # Forward local ports to sprite
```
These commands target the active sprite. Run `sprite use <name>` first if needed.

### 4. Surface results

Return command output directly. If a command fails, include the error output and suggest a remediation step (re-auth, check sprite name, verify org).

## Examples

**Create and exec:**
```
sprite create dev-sandbox
sprite exec -s dev-sandbox echo hello
```

**Auth recovery:**
```sh
# sprite list fails → re-authenticate
sprite auth setup --token <token>
# then retry the original command
```

**Checkpoint before a risky change:**
```
sprite checkpoint create -s dev-sandbox
# proceed with change
```

**Restore a checkpoint:**
```
sprite checkpoint list -s dev-sandbox
sprite restore <checkpoint-id>
```

**Get URL and make public:**
```sh
sprite use dev-sandbox
sprite url
sprite url update --auth public
```

**Destroy:**
```
sprite destroy dev-sandbox
```
