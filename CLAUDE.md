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

# OpenWrt + Xray Reality (transparent proxy)

## Skill

Skill `openwrt-xray-reality` is installed at `~/.claude/skills/openwrt-xray-reality/SKILL.md` вЂ” load it for any setup/repair of VLESS/Xray Reality transparent proxying on OpenWrt/GL.iNet routers (LuCI-based).

## Summary

- Validate VLESS Reality link(s) locally first, via a temporary Xray SOCKS5 inbound (`curl.exe --socks5-hostname 127.0.0.1:<port> https://www.google.com/generate_204` should return `HTTP 204`). Drop any link that fails before touching the router.
- The OpenWrt/GL.iNet package feed's `xray-core` (seen: 1.5.9) is too old for modern VLESS Reality/Vision вЂ” download a current official Xray release matching the router's CPU arch (check via LuCI `Status -> Overview` or `uname -m`).
- If SSH/Dropbear isn't reachable, package the binary/config/init script as a `.ipk` (gzip'd tar of `./debian-binary`, `./data.tar.gz`, `./control.tar.gz`) and install via LuCI `System -> Software -> Upload Package`.
- Xray config shape:
  - One `dokodemo-door` transparent TCP inbound (`followRedirect: true`, sniffing enabled for `http`/`tls`).
  - One VLESS Reality outbound per endpoint, tagged e.g. `proxy-<port>`.
  - A `leastPing` balancer across the `proxy-*` outbounds for automatic failover.
  - Avoid `geoip:private` unless `geoip.dat` is installed вЂ” use explicit private/reserved CIDRs for the direct-routing bypass instead.
- iptables NAT redirects LAN TCP (excluding private/reserved ranges) on `br-lan` into the Xray transparent inbound port.
- Caveats:
  - TCP only вЂ” UDP/QUIC bypasses the tunnel; some apps need QUIC disabled to avoid leaking traffic outside the tunnel.
  - Never store the router admin password in scripts/runbooks вЂ” enter it interactively in LuCI.
  - opkg may print errors from old/half-installed packages even when the new package installed and works fine.
