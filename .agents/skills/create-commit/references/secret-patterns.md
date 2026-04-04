# Secret Patterns Reference

Patterns to detect before staging files. If any match, block that file, report the finding, and suggest remediation.

## File-level exclusions

Skip the entire file if it matches any of these:

| Pattern | Reason |
|---------|--------|
| `.env`, `.env.*` | Environment variable files almost always contain secrets |
| `*.pem`, `*.key`, `*.p12`, `*.pfx` | Private key / certificate files |
| `credentials`, `credentials.json`, `secrets.json` | Common credential file names |
| `*_rsa`, `*_dsa`, `*_ecdsa`, `*_ed25519` | SSH private key files |

## Content patterns

Scan file content for these patterns (case-insensitive where noted):

### Explicit secret assignments

```
(API_KEY|SECRET_KEY|ACCESS_KEY|AUTH_TOKEN|PRIVATE_KEY|CLIENT_SECRET|DB_PASSWORD|DATABASE_URL)\s*=\s*\S+
```

Any variable assignment where the name strongly implies a credential and the value is non-empty.

### Cloud provider credentials

| Provider | Pattern |
|----------|---------|
| AWS access key | `AKIA[0-9A-Z]{16}` |
| AWS secret | `aws_secret_access_key\s*=\s*\S+` |
| GCP service account | `"type": "service_account"` inside a JSON file |
| Azure connection string | `DefaultEndpointsProtocol=https;AccountName=` |

### Private keys (PEM format)

```
-----BEGIN (RSA |EC |DSA |OPENSSH )?PRIVATE KEY-----
```

### High-entropy strings

Flag values that are 20+ characters of base64 or hex assigned to a variable whose name contains `key`, `token`, `secret`, `password`, `credential`, or `auth`:

```
(key|token|secret|password|credential|auth)\s*[=:]\s*["']?[A-Za-z0-9+/=]{20,}["']?
```

Use this as a heuristic — false positives are acceptable; it is better to flag and ask than to miss a real secret.

## Remediation guidance

When a pattern matches, report:

```
BLOCKED: <file path>
Reason: <pattern type> detected on line <N>
Fix: Remove the value from the file. Options:
  - Store in an environment variable and reference it as $VAR_NAME
  - Add the file to .gitignore if it must exist locally
  - Use a secrets manager (e.g. Vault, AWS Secrets Manager)
```

Do not print the matched secret value in the report.
