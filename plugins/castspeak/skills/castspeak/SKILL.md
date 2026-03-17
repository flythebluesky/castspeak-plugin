---
name: castspeak
description: Control Google Nest Mini and other Chromecast devices — speak text aloud, play audio URLs, scan for devices, set volume, mute/unmute, and check device status via the castspeak CLI. Use this skill whenever the user wants to interact with Cast devices, announce something on a speaker, play sounds on a Nest/Chromecast, or control smart speaker volume — even if they don't mention "castspeak" by name.
---

# CastSpeak Skill

Control Cast devices on the local network using the `castspeak` CLI.

## Prerequisites

Check if `castspeak` is available and up to date:

```bash
# Check if installed and show version
if command -v castspeak >/dev/null 2>&1; then
  INSTALLED="$(castspeak version)"
  LATEST="$(curl -fsSL -H 'Accept: application/vnd.github.v3+json' \
    https://api.github.com/repos/flythebluesky/castspeak/releases/latest \
    | grep '"tag_name"' | head -1 | cut -d'"' -f4)"
  echo "Installed: $INSTALLED  Latest: $LATEST"
  if [ "$INSTALLED" != "$LATEST" ] && [ "$INSTALLED" != "${LATEST#v}" ]; then
    echo "Update available — reinstall to get the latest version"
  fi
else
  echo "castspeak not found — install it below"
fi
```

If not found or needs updating, install it by running this script:

```bash
set -euo pipefail

# Detect OS and architecture
OS="$(uname -s | tr '[:upper:]' '[:lower:]')"
ARCH="$(uname -m)"
case "$ARCH" in
  x86_64)  ARCH="amd64" ;;
  aarch64) ARCH="arm64" ;;
  arm64)   ARCH="arm64" ;;
  *)       echo "Unsupported architecture: $ARCH" >&2; exit 1 ;;
esac

# Windows detection (Git Bash / MSYS / WSL)
case "$OS" in
  mingw*|msys*|cygwin*) OS="windows" ;;
esac

# Get latest release tag
LATEST="$(curl -fsSL -H 'Accept: application/vnd.github.v3+json' \
  https://api.github.com/repos/flythebluesky/castspeak/releases/latest \
  | grep '"tag_name"' | head -1 | cut -d'"' -f4)"

if [ -z "$LATEST" ]; then
  echo "Failed to fetch latest release" >&2; exit 1
fi

# Download and install
EXT="tar.gz"
[ "$OS" = "windows" ] && EXT="zip"
URL="https://github.com/flythebluesky/castspeak/releases/download/${LATEST}/castspeak_${OS}_${ARCH}.${EXT}"
INSTALL_DIR="${HOME}/.local/bin"
mkdir -p "$INSTALL_DIR"

echo "Downloading castspeak ${LATEST} for ${OS}/${ARCH}..."
TMPDIR="$(mktemp -d)"
cd "$TMPDIR"
curl -fsSL -o "castspeak.${EXT}" "$URL"

if [ "$EXT" = "zip" ]; then
  unzip -q "castspeak.${EXT}"
else
  tar xzf "castspeak.${EXT}"
fi

mv castspeak "$INSTALL_DIR/castspeak"
chmod +x "$INSTALL_DIR/castspeak"
rm -rf "$TMPDIR"

echo "Installed castspeak to $INSTALL_DIR/castspeak"
echo "Make sure $INSTALL_DIR is on your PATH"
```

After installation, verify it works:

```bash
export PATH="$HOME/.local/bin:$PATH"
castspeak --version
```

## Workflow

Discover devices before issuing commands. If mDNS is blocked (e.g. by CrowdStrike Falcon), use `scan` instead:

```bash
# Standard mDNS discovery
castspeak devices --timeout 5

# If mDNS is blocked, scan the network directly and save results
castspeak scan --timeout 15 --save
```

Then use the device name from the output in subsequent commands. When mDNS is blocked, saved devices are used automatically as a fallback.

## Commands

### Discover devices

```bash
# mDNS discovery (falls back to saved devices if mDNS blocked)
castspeak devices --timeout 5

# Network scan — finds devices via TCP port scan (no mDNS needed)
castspeak scan --timeout 15
castspeak scan --timeout 15 --save   # save results for offline use

# View/manage saved devices
castspeak devices saved
castspeak devices forget
```

### Speak text aloud

```bash
castspeak speak --device "<device name>" --text "<message>" --language en
```

### Play an audio URL

```bash
castspeak play --device "<device name>" --url "<media url>"
```

### Volume control

```bash
castspeak volume --device "<device name>" --level 0.5
castspeak mute --device "<device name>"
castspeak unmute --device "<device name>"
```

`--level` is a float: 0.0 (silent) to 1.0 (max). So 30% = `--level 0.3`.

### Stop playback

```bash
castspeak stop --device "<device name>"
```

### Check device status

```bash
castspeak status --device "<device name>"
```

### Start the HTTP API server

```bash
castspeak serve --port 8080
```

Starts a REST API on the given port (default 8080, or `PORT` env var). Useful for integrations that prefer HTTP over CLI.

## Targeting devices

- `--device "<name>"` — match by device name (from `castspeak devices` output)
- `--uuid "<uuid>"` — match by device UUID (also shown in discovery output)
- `--host "<ip:port>"` — connect directly by IP (skips discovery entirely, default port 8009)
- At least one is required for all commands except `devices`, `scan`, and `serve`.

## Tips

- **Discover first.** Always run `castspeak devices` or `castspeak scan` before other commands so you have the exact device name.
- **mDNS blocked?** Run `castspeak scan --save` once to save devices, then all commands will fall back to saved devices automatically.
- **Keep text short.** Text is split into ~200-char chunks — shorter messages play faster. Max 5000 characters.
- **Language codes.** `--language` accepts BCP-47 codes: `en`, `fr`, `es`, `de`, `ja`, etc. Default is `en`.
- **Timeout.** Discovery defaults to 5 seconds. If a device isn't found, try `--timeout 10`. Scan defaults to 10 seconds.
- **Direct connect.** Use `--host 192.168.1.42` to skip discovery entirely if you know the IP.
