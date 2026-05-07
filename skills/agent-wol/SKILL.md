---
name: agent-wol
description: Use when an agent or operator needs to wake a known machine with Wake-on-LAN using safe, user-provided network values and verify whether the target came online.
version: 1.0.0
author: p.a.t.h. @materializepath
license: MIT
metadata:
  hermes:
    tags:
      - devops
      - wake-on-lan
      - automation
      - network
    category: DevOps
    config:
      - key: agent_wol.target_name
        description: Friendly name for the machine or agent to wake.
        default: ""
        prompt: Target name
      - key: agent_wol.mac_address
        description: Target network adapter MAC address used for the WOL magic packet.
        default: ""
        prompt: Target MAC address
      - key: agent_wol.broadcast_address
        description: LAN broadcast address that can reach the target network segment.
        default: ""
        prompt: Broadcast address
      - key: agent_wol.ports
        description: Comma-separated UDP ports to try for WOL.
        default: "9,7"
        prompt: WOL UDP ports
      - key: agent_wol.verify_host
        description: Hostname or IP address to test after sending WOL packets.
        default: ""
        prompt: Host or IP to verify
required_environment_variables: []
---

# agent-WOL

## Overview

`agent-wol` is a public, generic Wake-on-LAN skill for waking a known machine or agent host from sleep or a supported low-power state. It keeps all machine-specific values out of the skill itself. Users provide their own MAC address, broadcast address, ports, and verification host at runtime or through Hermes configuration.

Wake-on-LAN is simple in concept: send a magic packet to the sleeping machine's network adapter. In practice, success depends on firmware settings, operating-system power settings, network routing, and whether the target NIC still has standby power.

## When to Use

Use this skill when:

- You need to wake a known desktop, workstation, server, media box, or agent host on a local network.
- You already know the target MAC address and a broadcast address that can reach the target LAN.
- You want to separate three things clearly: sending the magic packet, checking reachability, and diagnosing why wake did or did not work.

Do not use this skill for:

- Scanning a network to discover unknown devices.
- Bypassing network access controls.
- Waking machines on networks where you do not have permission.
- Assuming ping alone can wake a machine. Ping is only a diagnostic check.

## Required Inputs

The user must provide these values. Public examples must use placeholders only.

| Value | Meaning | Example placeholder |
| --- | --- | --- |
| Target name | Friendly label for the device | `<TARGET_NAME>` |
| MAC address | NIC address used in the magic packet | `<MAC_ADDRESS>` |
| Broadcast address | LAN broadcast address reachable from the sender | `<BROADCAST_ADDRESS>` |
| UDP ports | WOL ports to try | `9,7` |
| Verification host | Hostname or IP to test after wake | `<VERIFY_HOST>` |

## Safety Rules

- Never hardcode a real MAC address, LAN IP, VPN IP, hostname, username, or local path into the public skill.
- Do not print secrets or environment files.
- Do not assume a remembered address is current; verify when possible.
- Prefer subnet broadcast values supplied by the user over global broadcast.
- Send only a small number of WOL packets by default.
- Do not perform destructive operations. WOL should only send UDP packets and run non-invasive reachability checks.

## Wake Sequence

1. Confirm the target details:
   - target name
   - MAC address
   - broadcast address
   - UDP ports
   - verification host

2. Send a small burst of magic packets to each configured UDP port.

3. Wait briefly for the system to resume.

4. Verify reachability using safe checks:
   - ping the verification host
   - inspect ARP or neighbor table if useful
   - optionally test SSH, HTTP, or another service only if the user requests it

5. Report the result clearly:
   - packets sent or failed
   - verification replies or timeouts
   - whether a MAC mapping was visible
   - likely next diagnostic step

## Generic Python Sender

This snippet has no hardcoded private values and does not read secrets from the environment. Replace placeholders with user-provided values before running it.

```bash
python3 - <MAC_ADDRESS> <BROADCAST_ADDRESS> 9,7 3 <<'PY'
import socket
import time
import re
import sys

if len(sys.argv) != 5:
    sys.exit("usage: python3 sender.py <MAC_ADDRESS> <BROADCAST_ADDRESS> <PORTS> <RETRIES>")

mac = sys.argv[1].strip()
broadcast = sys.argv[2].strip()
ports_raw = sys.argv[3].strip()
retries_raw = sys.argv[4].strip()

if not re.fullmatch(r"[0-9A-Fa-f]{2}(:[0-9A-Fa-f]{2}){5}", mac):
    sys.exit("MAC address must have six hex byte pairs, such as <MAC_ADDRESS>")

if not broadcast or any(ch.isspace() for ch in broadcast):
    sys.exit("Broadcast address must be a single host or address, such as <BROADCAST_ADDRESS>")

try:
    ports = [int(p) for p in ports_raw.split(",") if p.strip()]
    retries = int(retries_raw)
except ValueError:
    sys.exit("Ports and retries must be numeric")

if not ports or any(p < 1 or p > 65535 for p in ports):
    sys.exit("Ports must be in the range 1-65535")

if retries < 1 or retries > 10:
    sys.exit("Retries must be between 1 and 10")

payload = bytes.fromhex("FF" * 6 + mac.replace(":", "") * 16)

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.setsockopt(socket.SOL_SOCKET, socket.SO_BROADCAST, 1)

for port in ports:
    for attempt in range(retries):
        sock.sendto(payload, (broadcast, port))
        print(f"sent magic packet to {broadcast}:{port} attempt {attempt + 1}/{retries}")
        time.sleep(0.25)
PY
```

## Verification Commands

Use placeholders only in shared docs or public examples.

```bash
ping -c 5 <VERIFY_HOST> || true
arp -an | grep '<VERIFY_HOST>' || true
```

On Linux, the neighbor table can be useful:

```bash
ip neigh show | grep '<VERIFY_HOST>' || true
```

Interpretation:

- A ping reply means the target is reachable now.
- A timeout does not prove the WOL packet failed; the target may still be booting or may block ping.
- An ARP or neighbor entry can show layer-2 memory, but it may be stale.
- If no checks succeed, the likely causes are firmware WOL disabled, NIC power disabled, OS power settings, wrong MAC, wrong broadcast address, or sending from the wrong network segment.

## Troubleshooting

| Symptom | Likely cause | Next step |
| --- | --- | --- |
| Magic packet sends but host never responds | Target is fully powered off or WOL is disabled | Check firmware and OS power settings locally |
| Broadcast send fails | Sender cannot route to that broadcast address | Use a LAN host or router on the correct subnet |
| Ping fails but device later wakes | Resume is delayed or ping is blocked | Wait longer and test the intended service |
| ARP shows a MAC but ping fails | ARP cache may be stale | Do not treat ARP alone as proof the target is awake |
| Works from one machine but not another | Broadcast path differs | Send from an always-on host on the target LAN |

## Public Release Hygiene

Before publishing a host-specific variant of this skill:

- Replace all real MAC addresses with `<MAC_ADDRESS>`.
- Replace all LAN or VPN addresses with placeholders.
- Replace hostnames with `<TARGET_NAME>` or `<VERIFY_HOST>`.
- Remove local usernames and absolute paths.
- Remove logs, sessions, credentials, and copied conversation snippets.
- Run local secret scans and review every finding before publishing.

## Verification Checklist

- [ ] Target MAC address is supplied by the user, not hardcoded in the public skill.
- [ ] Broadcast address is supplied by the user, not hardcoded in the public skill.
- [ ] Verification host is supplied by the user, not hardcoded in the public skill.
- [ ] No private IPs, VPN IPs, MAC addresses, usernames, or local paths are present.
- [ ] WOL packet sender validates inputs and limits retries.
- [ ] Verification steps are non-destructive.
- [ ] User approval is received before publishing or pushing anything.
