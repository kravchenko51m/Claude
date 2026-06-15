# Wiren Board

## Skills

WirenBoard PLC skills (`wiren-board`, `wb-troubleshooting`, `wb-serial`, `wb-rules`, `wb-mqtt-broker`, `wb-network`, `wb-zigbee`, `wb-controller-backup`, `wb-mod-slots`, `wb-dev`) are installed globally at `~/.claude/skills/`, sourced from `wb-plc/skills/` in https://github.com/wirenboard/wb-ai-skills.

The `wiren-board` skill auto-loads on any mention of Wiren Board / wb6/wb7/wb8 / `wb-*` tools and is the entry point (SSH conventions, `wb-cli` usage, links to the other skills).

To update: download https://github.com/wirenboard/wb-ai-skills/archive/refs/heads/main.zip, extract, and copy `wb-plc/skills/*` into `~/.claude/skills/`, overwriting existing folders.

## Controller: WB8.1 "AWHHWAPI"

Managed via wirenboard.cloud (account: kravchenko.a@gmail.com).

**Remote access status:** Only a browser-based web terminal is known so far, at `https://awhhwapi.ssh.wirenboard.cloud/?lang=en`. It requires an authenticated browser session on wirenboard.cloud and redirects to a login page otherwise - Claude has no browser automation tool and cannot drive it.

To get Claude direct SSH access, find a standalone `ssh -p <port> root@<relay>.wirenboard.cloud`-style connection string in the WB Cloud dashboard (separate from the web terminal link), or set up a VPN/Tailscale/WireGuard tunnel to the controller's network.

Default controller SSH credentials: `root` / `wirenboard` (unless changed).