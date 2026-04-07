# Sprite CLI Cheat Sheet

Reference for the `sprite` CLI (external instance management). Run `sprite --help` or `sprite <command> --help` for full options.

## Authentication

```sh
sprite login                          # Browser-based login (Fly.io)
sprite auth setup --token <token>     # Token-based login (no browser)
sprite org auth                       # Org token flow
sprite logout                         # Remove local config
```

## Instance Lifecycle

```sh
sprite create <name>                  # Create a new sprite
sprite create -o <org> <name>         # Create in a specific org
sprite list                           # List all sprites (alias: ls)
sprite list -o <org>                  # List sprites in an org
sprite use <name>                     # Activate sprite for current directory
sprite use --unset                    # Clear active sprite
sprite destroy <name>                 # Destroy a sprite (irreversible)
sprite destroy -o <org> <name>
```

## Execution

```sh
sprite exec <command> [args...]       # Run command in active sprite
sprite exec -s <name> <command>       # Run command in named sprite
sprite -o <org> -s <name> exec <cmd>  # Scoped to org and sprite
sprite console                        # Interactive shell (active sprite)
sprite console -s <name>              # Interactive shell (named sprite)
```

## Checkpoints

```sh
sprite checkpoint create              # Snapshot active sprite
sprite checkpoint create -s <name>    # Snapshot named sprite
sprite checkpoint list                # List checkpoints (alias: ls)
sprite checkpoint list -s <name>
sprite checkpoint info <id>           # Details for a specific checkpoint
sprite restore <id>                   # Restore from checkpoint (drops session!)
```

## Networking & URL

```sh
sprite url                            # Show public URL for active sprite
sprite url update --auth public       # Make URL public (open to internet)
sprite url update --auth sprite       # Restrict to org members (default)
sprite proxy <port> [port2...]        # Forward local ports to sprite
```

## Flags

| Flag | Effect |
|------|--------|
| `-s <name>` | Target sprite by name |
| `-o <org>` | Target org |
| `--debug` | Debug logging to stdout |
| `--debug=<file>` | Debug logging to file |

## Environment Variables

| Variable | Effect |
|----------|--------|
| `SPRITE_VERSION_DEBUG=true` | Show detailed version check info |
| `UPGRADE_CHECK=true` | Force version check (bypass 24-hour cache) |

## Required Auth

Authentication is handled out of band. Run `sprite login` or `sprite auth setup --token <token>` before issuing any other commands. Tokens can be scoped to an org via `sprite org auth`.
