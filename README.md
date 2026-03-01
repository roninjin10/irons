# irons

`irons` is a CLI tool for spinning up egress-secured cloud VMs (sandboxes) designed for use with AI agents. It lets you create isolated, SSH-accessible environments with fine-grained control over outbound network traffic.

## Get Access

**We're currently in early access.** [Schedule a call →](https://cal.com/matthew-slipper-ironcd/15min) and we'll get you set up with API keys in 15 minutes.

## Installation

### Install Script (recommended)

```sh
curl -fsSL https://raw.githubusercontent.com/ironsh/irons/main/install.sh | bash
```

### Download Binary

Pre-built binaries for macOS and Linux are available on the [GitHub Releases](https://github.com/ironsh/irons/releases/latest) page.

### From Source

Requires Go 1.24+.

```sh
git clone https://github.com/ironsh/irons.git
cd irons
go install github.com/ironsh/irons@latest
```

## Authentication

All commands require an API key. Set it via environment variable (recommended):

```sh
export IRONS_API_KEY=your-api-key
```

Or pass it directly with `--api-key`. By default, `irons` talks to `https://elrond.ironcd.com` — override with `IRONS_API_URL` or `--api-url`.

## Quick Start

```sh
# Create a sandbox and wait until it's ready
irons create my-sandbox

# SSH in
irons ssh my-sandbox

# Tear it down when done
irons destroy my-sandbox
```

Commands accept either a sandbox **name** or its **VM ID** (e.g. `vm_abc123`) — whichever is more convenient.

## Documentation

Full command reference, egress configuration, and guides are at **[docs.iron.sh](https://docs.iron.sh)**.

## License

See [LICENSE](LICENSE).
