# agent-WOL

Public Hermes skill for safe, generic Wake-on-LAN workflows.

I originally created this because I did not want my local LLM / GPU workstation running 24/7. If that system was asleep, I needed a way to remotely wake it so I could use local LLMs or a graphics/CUDA-focused agent on that machine.

With this skill, a user can message a main agent with something like: “wake up Agent Hawk,” and the agent can send Wake-on-LAN packets to the right machine, then verify whether it came online.

Author: p.a.t.h. @materializepath
License: MIT

## What it does

`agent-wol` helps a Hermes agent wake a known machine by sending Wake-on-LAN magic packets with user-provided values. It can also guide users through safe discovery of the target adapter MAC address and the sender LAN broadcast address. The skill keeps verification and troubleshooting separate so users can tell the difference between packet delivery, network reachability, and actual machine wake state.

## What it does not do

- It does not include any real hostnames, IP addresses, MAC addresses, usernames, tokens, or local paths.
- It does not broad-scan networks for unknown devices.
- It does not inventory networks without permission.
- It does not bypass network permissions.
- It does not publish or store credentials.
- It does not guarantee wake success if firmware, operating-system, or NIC power settings block WOL.

## Requirements

- Hermes Agent installed.
- A sender machine on a network path that can reach the target LAN broadcast address.
- The target machine's Wake-on-LAN support enabled in firmware and operating-system power settings.
- The target network adapter MAC address, or permission to help discover it while the target is awake.
- A broadcast address for the target LAN, or the sender LAN interface details needed to calculate it.

## Environment variables

No secret environment variables are required.

The example sender in `SKILL.md` uses command-line placeholders rather than environment variables, so users can keep private machine details in their own notes, private Hermes config, or shell history policy as they prefer.

## Configuration values

The skill frontmatter defines optional Hermes configuration prompts for:

- `agent_wol.target_name`
- `agent_wol.mac_address`
- `agent_wol.broadcast_address`
- `agent_wol.ports`
- `agent_wol.verify_host`

Use placeholders in public examples and your own real values only in your private Hermes config or runtime environment.

## Discovery help

If a user does not know the MAC address or broadcast address yet, ask Hermes:

```text
Use agent-wol to help me discover the MAC address and broadcast address for <TARGET_NAME>. I know the target host as <TARGET_HOST> and my sender interface is <INTERFACE_NAME>.
```

The skill uses narrow, permission-based checks:

- read adapter addresses on the target machine if the target is awake
- query ARP or neighbor tables for a known target host
- read the sender interface broadcast value from the OS
- calculate a broadcast address from `<LAN_IP>/<PREFIX_LENGTH>` when supplied

It should not broad-scan an unknown network by default.

## Usage examples

Ask Hermes:

```text
Use agent-wol to wake <TARGET_NAME> using MAC <MAC_ADDRESS>, broadcast <BROADCAST_ADDRESS>, ports 9 and 7, then verify <VERIFY_HOST>.
```

Manual sender example:

```bash
python3 - <MAC_ADDRESS> <BROADCAST_ADDRESS> 9,7 3 <<'PY'
# Use the complete sender from SKILL.md.
PY
```

## Installation

Add the repository as a Hermes skills tap:

```bash
hermes skills tap add materializepath/hermes-public-skills
hermes skills search agent-wol --source github
hermes skills install materializepath/hermes-public-skills/skills/agent-wol
```

You can inspect the skill directly:

```bash
hermes skills inspect materializepath/hermes-public-skills/skills/agent-wol
```

## Troubleshooting

- If packets send but the machine does not wake, check firmware WOL settings, NIC standby power, and OS power settings.
- If the broadcast send fails, re-check the sender interface broadcast address or send from a machine on the correct LAN segment.
- If the discovered MAC does not work, verify the adapter on the target machine or router while the target is awake.
- If ping fails after wake, wait longer or test the actual service you expect to become available.
- If ARP shows a MAC but the target does not respond, treat the ARP entry as possible stale cache.

## License

MIT
