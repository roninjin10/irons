# VM Specs & Pre-installed Tools

## Hardware

| Spec | Value |
|------|-------|
| vCPUs | 4 |
| RAM | 8 GB |
| Disk | 64 GB |
| OS | Ubuntu 24.04 LTS (Noble) |

Higher resources available upon request.

## APT Configuration

Default mirror: `mirrors.kernel.org` (instead of standard Ubuntu mirrors).

## Pre-installed System Packages

**Core build tools:** build-essential, cmake, pkg-config, autoconf, automake, libtool

**Version control:** git, git-lfs

**Editors & multiplexers:** vim, neovim, tmux, screen

**Networking & debugging:** curl, wget, net-tools, iproute2, dnsutils, traceroute, mtr, tcpdump, nmap, openssh-client, openssh-server, socat

**JSON & data tools:** jq, xmlstarlet, sqlite3

**Compression:** zip, unzip, tar, gzip, bzip2, xz-utils, p7zip-full, rsync, zstd

**System tools:** htop, btop, strace, ltrace, lsof, sysstat, tree, fd-find, ripgrep, fzf

**Security:** openssl, ca-certificates, gnupg

**Development libraries:** libssl-dev, libffi-dev, zlib1g-dev, libreadline-dev, libsqlite3-dev, libcurl4-openssl-dev

**Misc:** software-properties-common, apt-transport-https, locales, sudo, less, file, patch, direnv, shellcheck, man-db

## Runtime Environments

| Runtime | Details |
|---------|---------|
| Docker | Engine installed and available |
| Python | python3, python3-pip, python3-venv, python3-dev, python3-setuptools, python3-wheel, pipx |
| Node.js | Managed via fnm, latest LTS pre-installed |
| Go | Installed to `/usr/local/go`, GOPATH configured |
| Ruby | Version 3 installed via mise |
| Rust | Installed via rustup for the `ubuntu` user |

## Additional Tools

GitHub CLI (`gh`), just, OpenAI Codex, Claude Code, mise, Foundry (forge, cast, anvil, chisel)

## PATH Configuration

The `ubuntu` user's `.bashrc` includes:

```
/usr/local/go/bin:$HOME/.local/bin:$HOME/.foundry/bin:$HOME/.local/share/fnm:$HOME/.cargo/bin
```

mise and fnm shell integrations are enabled.
