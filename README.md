# Agent Plugins

Agent Plugins is an open, vendor-neutral specification for packaging agent extensions into distributable plugins. It defines a portable package format for Agent Skills and MCP servers.

This README is a non-normative introduction. The versioned specification defines the portable contract.

## Status

Agent Plugin Specification 1.0.0 is a working draft and has not been published.

## Quick Start

The smallest useful plugin is a directory with one skill:

```text
hello-plugin/
├── plugin.json
└── skills/
    └── greet/
        └── SKILL.md
```

In `plugin.json`:

```json
{
  "$schema": "https://agent-plugins.org/schemas/1.0.0/plugin.schema.json",
  "name": "hello-plugin"
}
```

In `skills/greet/SKILL.md`:

```markdown
---
name: greet
description: Greet the user and offer help.
---

Greet the user and offer help.
```

A host that supports skills can load the plugin by reading `plugin.json` and discovering `skills/greet/SKILL.md`. How the host exposes the skill to users or models is outside the Agent Plugin specification.

## Project Documents

- [Agent Plugin Specification 1.0.0 working draft](./spec/1.0.0.md)
- [Plugin manifest schema](./schemas/1.0.0/plugin.schema.json)
- [MCP configuration schema](./schemas/1.0.0/mcp.schema.json)
- [Technical Charter](./GOVERNANCE.md)
- [Future considerations](./FUTURE_CONSIDERATIONS.md)
