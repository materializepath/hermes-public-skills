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

## License

Skills in this repository are published under the MIT License unless a specific skill says otherwise.
