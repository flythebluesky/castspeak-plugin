# castspeak-plugin

Claude Code plugin for controlling Google Nest Mini and Chromecast devices via [castspeak](https://github.com/flythebluesky/castspeak).

## Install

```
/plugin install castspeak@flythebluesky/castspeak-plugin
```

## Prerequisites

You need the `castspeak` binary built and on your PATH. From the [castspeak repo](https://github.com/flythebluesky/castspeak):

```bash
go build -o castspeak .
```

## What it does

Once installed, Claude Code gains a `castspeak` skill that lets you control Cast devices with natural language:

- "announce dinner on my Nest Mini"
- "play this URL on the living room speaker"
- "set the bedroom speaker to 30%"
- "what's playing on the kitchen display?"

The skill handles device discovery, text-to-speech, audio playback, volume control, and status checks.