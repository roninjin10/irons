# Sandbox Model & Egress Architecture

## Compute Isolation

Every sandbox runs in its own dedicated virtual machine. Sandboxes cannot communicate with each other or with Iron.sh infrastructure. Data persists only if explicitly copied out — filesystems are destroyed upon teardown.

## Network Bridge Architecture

All outbound traffic from a sandbox is routed through a network bridge that Iron.sh controls. The bridge sits between the sandbox and the internet and is responsible for observing and enforcing egress rules.

## Traffic Rules

| Traffic Type | Behavior |
|---|---|
| Inbound SSH (TCP 22) | Allowed via bridge from host to VM |
| Outbound SSH (TCP 22) | Allowed only to manually whitelisted IP ranges |
| ICMP (ping) | Allowed |
| HTTPS / other TCP | Allowed or blocked based on egress rules |
| All other traffic | Blocked |

## Egress Modes

| Mode | Behavior |
|---|---|
| warn | All outbound connections permitted, but violations are logged |
| enforce | Outbound connections to non-allowlisted domains are blocked |

## Outbound SSH

The SSH client resolves DNS before opening the TCP connection, so by the time the bridge sees the traffic there is no hostname to match against — only an IP address. Outbound SSH therefore requires IP/CIDR whitelisting, not domain-based rules.

```bash
irons egress add --cidr 140.82.112.0/20 --name "github-ssh" --comment "GitHub SSH access"
```

## Default Egress Allowlist

New accounts include rules for common package managers, container registries, and AI services. Use `irons egress list` to see the full set. Categories include:

**GitHub:** github.com, ghcr.io, *.actions.githubusercontent.com, and related API/package/container hosts

**APT:** mirrors.edge.kernel.org and related Debian/Ubuntu mirrors

**Node.js:** registry.npmjs.org, registry.yarnpkg.com, nodejs.org, and related hosts

**PyPI:** pypi.org, *.pythonhosted.org, and test/upload endpoints

**Go:** proxy.golang.org, sum.golang.org, pkg.go.dev, and related hosts

**Rust:** crates.io, index.crates.io, sh.rustup.rs, and related hosts

**Ruby:** rubygems.org, index.rubygems.org, cache.ruby-lang.org, and related hosts

**Java:** repo.maven.apache.org, repo1.maven.org

**Docker:** registry-1.docker.io, auth.docker.io, quay.io, and related registries

**Ubuntu:** security.ubuntu.com, *.archive.ubuntu.com, and infrastructure hosts

**OpenAI:** api.openai.com, chatgpt.com, and related hosts

**Anthropic:** api.anthropic.com, claude.ai, and related hosts

**Homebrew:** formulae.brew.sh

**Foundry:** foundry.paradigm.xyz

**Misc:** mise.run, tuf-repo-cdn.sigstore.dev, www.example.com, and others

**GitHub SSH CIDRs:** A set of GitHub IP ranges for Git-over-SSH operations.

## Egress Observation

The bridge inspects hostnames from TLS SNI and HTTP Host headers, recording all connection attempts in audit logs accessible via `irons audit egress`. In warn mode, blocked-by-policy connections are logged with verdict `warn` but still allowed through. In enforce mode, they are logged with verdict `blocked` and the connection is dropped.
