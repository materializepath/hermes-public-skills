# agent-WOL

Public Hermes skill for safe, generic Wake-on-LAN workflows.

Author: p.a.t.h. @materializepath
License: MIT

## What it does

`agent-wol` helps a Hermes agent wake a known machine by sending Wake-on-LAN magic packets with user-provided values. It also provides a careful verification and troubleshooting sequence so users can tell the difference between packet delivery, network reachability, and actual machine wake state.

## What it does not do

- It does not include any real hostnames, IP addresses, MAC addresses, usernames, tokens, or local paths.
- It does not scan networks for unknown devices.
- It does not bypass network permissions.
- It does not publish or store credentials.
- It does not guarantee wake success if firmware, operating-system, or NIC power settings block WOL.

## Requirements

- Hermes Agent installed.
- A sender machine on a network path that can reach the target LAN broadcast address.
- The target machine's Wake-on-LAN support enabled in firmware and operating-system power settings.
- The target network adapter MAC address.
- A broadcast address for the target LAN.

## Environment variables

No secret environment variables are required.

The example sender in `skills/agent-wol/SKILL.md` uses command-line placeholders rather than environment variables, so users can keep private machine details in their own notes, private Hermes config, or shell history policy as they prefer.

## Configuration values

The skill frontmatter defines optional Hermes configuration prompts for:

- `agent_wol.target_name`
- `agent_wol.mac_address`
- `agent_wol.broadcast_address`
- `agent_wol.ports`
- `agent_wol.verify_host`

Use placeholders in public examples and your own real values only in your private Hermes config or runtime environment.

## Usage examples

Ask Hermes:

```text
Use agent-wol to wake <TARGET_NAME> using MAC <MAC_ADDRESS>, broadcast <BROADCAST_ADDRESS>, ports 9 and 7, then verify <VERIFY_HOST>.
```

Manual sender example:

```bash
python3 - <MAC_ADDRESS> <BROADCAST_ADDRESS> 9,7 3 <<'PY'
# Use the complete sender from skills/agent-wol/SKILL.md.
PY
```

## Installation

After this repo is published as a Hermes skill source, users can add it as a tap:

```bash
hermes skills tap add materializepath/hermes-public-skills
hermes skills search agent-wol
hermes skills install materializepath/hermes-public-skills/agent-wol
```

If direct path install is more appropriate for this repo layout, use:

```bash
hermes skills install materializepath/hermes-public-skills/skills/agent-wol
```

## Publishing command

The intended official Hermes Skills Hub publish command is:

```bash
hermes skills publish skills/agent-wol --to github --repo materializepath/hermes-public-skills
```

Do not run that command until the skill has been sanitized, scanned, reviewed, and explicitly approved for release.

## Security notes

- Keep real MAC addresses, LAN addresses, VPN addresses, and hostnames out of public repos.
- Store private target values in private Hermes configuration, local shell environment variables, or another private secret/config mechanism.
- Do not commit `.env`, `config.yaml`, `auth.json`, logs, sessions, keys, tokens, or credential files.
- Run gitleaks before publishing.

## Troubleshooting

- If packets send but the machine does not wake, check firmware WOL settings, NIC standby power, and OS power settings.
- If the broadcast send fails, send from a machine on the correct LAN segment.
- If ping fails after wake, wait longer or test the actual service you expect to become available.
- If ARP shows a MAC but the target does not respond, treat the ARP entry as possible stale cache.

## License

MIT
