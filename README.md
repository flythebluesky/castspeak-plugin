# castspeak-plugin

Claude Code plugin for controlling Google Nest Mini and Chromecast devices via [castspeak](https://github.com/flythebluesky/castspeak).

## Install

```
/plugin marketplace add flythebluesky/castspeak-plugin
/plugin install castspeak@castspeak
```

## What it does

Once installed, Claude Code gains a `castspeak` skill that lets you control Cast devices with natural language. The `castspeak` binary is automatically downloaded from [GitHub Releases](https://github.com/flythebluesky/castspeak/releases) on first use — no Go toolchain required.

- "announce dinner on my Nest Mini"
- "play this URL on the living room speaker"
- "set the bedroom speaker to 30%"
- "what's playing on the kitchen display?"

The skill handles device discovery, text-to-speech, audio playback, volume control, and status checks.