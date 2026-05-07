# Hermes Public Skills

Public, sanitized Hermes skills by p.a.t.h. @materializepath.

This repository is a small collection of reusable Hermes Agent skills. Each skill lives in its own folder under `skills/` and includes a `SKILL.md` file for Hermes plus optional supporting docs.

## Available skills

| Skill | Description | Path |
| --- | --- | --- |
| `agent-wol` | Wake a known machine or agent host with Wake-on-LAN, including safe discovery of the target MAC address and LAN broadcast address. | `skills/agent-wol/` |

## Repository layout

```text
hermes-public-skills/
├── README.md
├── .gitignore
├── .github/
│   └── workflows/
│       └── gitleaks.yml
└── skills/
    └── agent-wol/
        ├── README.md
        └── SKILL.md
```

## Install from this repository

Add this repository as a Hermes skills tap:

```bash
hermes skills tap add materializepath/hermes-public-skills
hermes skills search agent-wol
hermes skills install materializepath/hermes-public-skills/skills/agent-wol
```

You can also inspect the skill directly:

```bash
hermes skills inspect materializepath/hermes-public-skills/skills/agent-wol
```

## Notes for contributors

- Keep each skill self-contained under `skills/<skill-name>/`.
- Public skill examples should use placeholders instead of private machine details.
- Do not commit secrets, local config, logs, sessions, or credentials.
- This repository runs gitleaks on pushes and pull requests.

## License

Skills in this repository are published under the MIT License unless a specific skill says otherwise.
