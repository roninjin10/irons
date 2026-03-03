---
name: irons
description: >-
  Reference for the irons CLI tool, which manages egress-secured cloud VMs
  (sandboxes) for AI agents. Use when the user mentions irons, sandboxes,
  egress rules, or cloud VMs for agents.
allowed-tools: Bash(irons *)
---

# irons CLI Reference

irons spins up egress-secured cloud VMs (sandboxes) for AI agents. Each sandbox is an isolated Ubuntu 24.04 VM with 4 vCPUs, 8 GB RAM, and 64 GB disk. All outbound traffic passes through a network bridge that observes and enforces egress rules.

## Authentication

Three methods, checked in order:

1. `--api-key` flag
2. `IRONS_API_KEY` environment variable
3. `~/.config/irons/config.yml` (`api_key: <key>`)

Run `irons login` to authenticate via browser and save the key to the config file.

## Global Flags

| Flag | Env Var | Default | Description |
|------|---------|---------|-------------|
| `--api-key` | `IRONS_API_KEY` | — | API key for authentication |
| `--api-url` | `IRONS_API_URL` | `https://api.iron.sh/v1` | API endpoint URL |
| `--debug-api` | `IRONS_DEBUG_API` | `false` | Dump API requests/responses to stderr |

## Sandbox Addressing

Commands accept a sandbox **name** or **VM ID** (prefixed `vm_`). Values without `vm_` are treated as names.

---

## Commands

### login

Opens browser-based device authorization flow. Saves API key to `~/.config/irons/config.yml`.

```
irons login
```

### create

```
irons create <name> [flags]
```

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--key` | `-k` | First key in `~/.ssh` | SSH public key path |
| `--async` | | `false` | Return immediately without waiting |

Key search order: `id_ed25519.pub`, `id_ed25519_sk.pub`, `id_ecdsa.pub`, `id_ecdsa_sk.pub`, `id_rsa.pub`.

### list

```
irons list
```

Displays a table of every sandbox with name, ID, status, and creation date.

### status

```
irons status <name|id>
```

Shows current status with indicators: 🟢 running, 🟡 starting, 🟠 stopped, 🔴 error.

### start

```
irons start <name|id> [flags]
```

| Flag | Default | Description |
|------|---------|-------------|
| `--async` | `false` | Return immediately without waiting |

### stop

```
irons stop <name|id> [flags]
```

| Flag | Default | Description |
|------|---------|-------------|
| `--async` | `false` | Return immediately without waiting |

### destroy

```
irons destroy <name|id> [flags]
```

| Flag | Default | Description |
|------|---------|-------------|
| `--force` | `false` | Stop the VM first if currently running |

### ssh

```
irons ssh <name|id> [flags]
```

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--command` | `-c` | `false` | Print SSH command instead of executing |
| `--strict-hostkeys` | | `false` | Enable strict host key checking |

### scp

```
irons scp <src> <dst> [flags]
```

Use `<name>:<path>` syntax for remote paths (e.g. `my-sandbox:/tmp/file.txt`).

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--recursive` | `-r` | `false` | Recursively copy directories |
| `--command` | `-c` | `false` | Print SCP command instead of executing |
| `--strict-hostkeys` | | `false` | Enable strict host key checking |

### forward

```
irons forward <name|id> [flags]
```

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--remote-port` | `-r` | — | **Required.** Remote port to forward |
| `--local-port` | `-l` | same as remote | Local port to listen on |
| `--command` | `-c` | `false` | Print SSH command instead of executing |
| `--strict-hostkeys` | | `false` | Enable strict host key checking |

---

## Egress Management

### egress list

```
irons egress list
```

Lists all current egress rules for the account.

### egress add

```
irons egress add [flags]
```

| Flag | Description |
|------|-------------|
| `--host` | Hostname to allow (mutually exclusive with `--cidr`) |
| `--cidr` | CIDR range to allow (mutually exclusive with `--host`) |
| `--name` | Optional human-readable name |
| `--comment` | Optional comment |

### egress remove

```
irons egress remove <rule_id>
```

### egress mode

```
irons egress mode [enforce|warn]
```

Without arguments, prints the current mode. With an argument, sets it.

- **warn** — all outbound connections permitted, violations logged
- **enforce** — non-allowlisted connections blocked

---

## Audit

### audit egress

```
irons audit egress [flags]
```

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--follow` | `-f` | `false` | Continuously poll for new events (like `tail -f`) |
| `--vm` | | — | Filter by sandbox name or VM ID |
| `--verdict` | | — | Filter: `allowed`, `blocked`, `warn` |
| `--since` | | 1 hour ago | Events after this timestamp (RFC 3339) |
| `--until` | | — | Events before this timestamp (RFC 3339) |
| `--limit` | | `0` | Max events to return |

Output fields (space-separated): `TIMESTAMP`, `VERDICT`, `VM_ID`, `PROTOCOL`, `HOST`, `MODE`.

---

## Common Workflows

### Quickstart

```bash
irons login
irons create my-sandbox
irons ssh my-sandbox
# ... work inside the VM ...
irons destroy my-sandbox --force
```

### Configure Egress

```bash
irons egress add --host api.github.com --name "github-api"
irons egress add --cidr 10.0.0.0/8 --name "internal" --comment "Internal network"
irons egress mode enforce
```

### Audit-then-Enforce

```bash
irons egress mode warn
irons audit egress --vm my-sandbox --follow
# observe traffic, add needed rules
irons egress mode enforce
```

### Copy Files

```bash
irons scp ./local-file.txt my-sandbox:/tmp/file.txt     # upload
irons scp my-sandbox:/tmp/file.txt ./local-file.txt     # download
irons scp -r ./my-dir my-sandbox:/home/ubuntu/my-dir    # upload directory
```

### Port Forwarding

```bash
irons forward my-sandbox -r 8080              # forward remote 8080 to local 8080
irons forward my-sandbox -r 3000 -l 9000      # forward remote 3000 to local 9000
```

---

## Supporting Reference

For deeper details, see:

- [REST API Reference](./reference/api.md)
- [Sandbox Model & Egress Architecture](./reference/sandbox-model.md)
- [VM Specs & Pre-installed Tools](./reference/vm-specs.md)
- [Troubleshooting](./reference/troubleshooting.md)
