<div align="center">

# hl-claude-plugins

**Claude Code skills, shipped as installable plugins.**

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-plugin-d97757)](https://docs.claude.com/en/docs/claude-code)

[Install](#install) · [Plugins](#plugins) · [中文](README.zh.md)

</div>

---

This repository is a **plugin marketplace**: one repo that holds a growing set of plugins, each bundling one or more skills. Add the marketplace once, then install whichever plugins you want.

## Requirements

- **[Claude Code](https://docs.claude.com/en/docs/claude-code)**

## Install

```bash
# 1. Add this marketplace
/plugin marketplace add hleidev/hl-claude-plugins

# 2. Install a plugin
/plugin install research@hl-claude-plugins
/plugin install java@hl-claude-plugins
```

## Plugins

| Plugin | Skills | Invoke | What it does |
|---|---|---|---|
| `research` | `research` | `/research:research` | Structured external-information research that produces a decision-grade report |
| `java` | `review` | `/java:review` | Enterprise Java/Spring code review against Alibaba P3C and Google Engineering Practices |

> Both plugins default to **Chinese** output. To use another language, change the "Output language" setting in the skill's `SKILL.md`, or just ask in your own language when you invoke it.

More plugins will be added to this same repo over time — adding one never requires a new repository.

## Repository layout

```
hl-claude-plugins/
├── .claude-plugin/
│   └── marketplace.json          # registry: lists every plugin below
├── plugins/
│   ├── research/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json        # plugin manifest
│   │   └── skills/
│   │       └── research/
│   │           └── SKILL.md       # the skill
│   └── java/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       └── skills/
│           └── review/
│               └── SKILL.md
├── README.md
├── README.zh.md
└── LICENSE
```

## License

MIT — see [LICENSE](LICENSE).
