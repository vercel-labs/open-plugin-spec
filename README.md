# Open Plugin Specification

**Spec Version: 1.0.0**

This document defines the canonical Open Plugin Specification v1.0.0. It is a self-contained specification for packaging agent extensions into distributable plugins. Everything in sections 1–12 is required for conformance.

## Quick Start (for context only — not required for conformance)

The smallest useful plugin is a directory with one skill.

```text
hello-plugin/
├── .plugin/
│   └── plugin.json
└── skills/
    └── greet/
        └── SKILL.md
```

`./.plugin/plugin.json`

```json
{
  "name": "hello-plugin"
}
```

`./skills/greet/SKILL.md`

```markdown
---
name: greet
description: Greet the user and offer help.
---

Greet the user. If `$ARGUMENTS` is present, include it in the greeting.
```

A host that supports skills can load this plugin by reading `.plugin/plugin.json`, discovering `skills/greet/SKILL.md`, and surfacing `/hello-plugin:greet`.

> **Note:**
> The Core Profile reading path in this document is package layout (§4), manifest loading (§5–6), discovery (§7), skills and MCP servers (§8), namespacing (§9), `${PLUGIN_ROOT}` expansion (§10), and minimum host conformance (§12). Commands, agents, rules, hooks, LSP servers, and output styles are optional extended component types defined in Appendix E.

## Table of contents

1. [Status and version](#1-status-and-version)
2. [Conformance language](#2-conformance-language)
3. [Terminology](#3-terminology)
4. [Plugin package model](#4-plugin-package-model)
5. [Manifest location and precedence](#5-manifest-location-and-precedence)
6. [Manifest schema](#6-manifest-schema)
7. [Component discovery](#7-component-discovery)
8. [Component definitions](#8-component-definitions)
9. [Namespacing](#9-namespacing)
10. [Environment variables and placeholder expansion](#10-environment-variables-and-placeholder-expansion)
11. [Versioning](#11-versioning)
12. [Host conformance](#12-host-conformance)

**Appendices (not required for conformance)**

- [Appendix A: Conformance Checklist](#appendix-a-conformance-checklist)
- [Appendix B: Marketplace Index and Discovery](#appendix-b-marketplace-index-and-discovery)
- [Appendix C: User Configuration](#appendix-c-user-configuration)
- [Appendix D: Extended Hook Events](#appendix-d-extended-hook-events)
- [Appendix E: Extended Component Types](#appendix-e-extended-component-types)
- [Design Decisions](#design-decisions)
- [Future Considerations](#future-considerations)

## 1. Status and version

This specification defines version `1.0.0` of the Open Plugin format.

Plugin hosts and plugin packages claiming conformance to Open Plugin v1 MUST implement or follow the requirements in this document.

## 2. Conformance language

The key words MUST, MUST NOT, REQUIRED, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL in sections 1–12 of this document are to be interpreted as described in RFC 2119 and RFC 8174 when, and only when, they appear in all capitals. Appendices and other non-normative sections use these terms informally.

## 3. Terminology

| Term               | Meaning                    | Description                                                                                                                                  |
| ------------------ | -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| Plugin             | Package unit               | A self-contained directory that bundles one or more components and optional metadata.                                                        |
| Plugin root        | Filesystem root            | The top-level directory of a plugin package.                                                                                                 |
| Manifest           | Metadata document          | A `plugin.json` file in a metadata directory at the plugin root.                                                                             |
| Component          | Plugin-provided capability | A skill or MCP server configuration (core), or an extended type such as a command, agent, rule, hook, LSP server, or output style.            |
| Host               | Plugin runtime             | A tool that discovers, installs, loads, and executes plugin components.                                                                      |
| Discovery source   | Scan location              | A default location, manifest-declared path, inline configuration object, or marketplace entry from which a host loads components.            |
| Path config        | Discovery object           | An object with `paths` that controls scanning for a component type.                                                                          |
| Inline MCP config  | MCP definition object      | A `mcpServers` manifest field whose object value contains an `mcpServers` key.                                                               |
| Marketplace        | Plugin collection          | A named collection of one or more plugins declared by `marketplace.json`.                                                                    |

## 4. Plugin package model

### 4.1 General requirements

1. A plugin is a directory rooted at a single filesystem location.
2. A plugin MUST include a manifest at `.plugin/plugin.json`.
3. A plugin MUST contain zero or more supported components. A directory with only a manifest is valid but may not be useful on hosts that require at least one supported component at runtime.
4. All relative paths declared by the plugin MUST be interpreted relative to the plugin root.
5. All relative paths declared by the plugin MUST start with `./`.
6. A plugin MUST NOT reference files outside its own directory tree by using `../` traversal.
7. Hosts MUST reject any configured path that escapes the plugin root after path normalization.

Example: valid and invalid relative paths

```json
{
  "skills": "./custom-skills/",
  "mcpServers": "./config/mcp.json"
}
```

```json
{
  "skills": "../shared-skills/",
  "mcpServers": "config/mcp.json"
}
```

The first example is valid — all paths start with `./` and stay within the plugin root. The second is invalid — `../shared-skills/` escapes the plugin root and `config/mcp.json` does not start with `./`.

### 4.2 Standard layout

```text
my-plugin/
├── .plugin/
│   └── plugin.json
├── commands/
├── agents/
├── skills/
├── output-styles/
├── rules/
├── hooks/
│   └── hooks.json
├── .mcp.json
├── .lsp.json
├── scripts/
├── assets/
├── LICENSE
└── CHANGELOG.md
```

Example: Core Profile layout (minimal)

```text
code-assistant/
├── .plugin/
│   └── plugin.json
├── skills/
│   └── summarize/
│       └── SKILL.md
└── .mcp.json
```

Example: full plugin with all component types

```text
devtools/
├── .plugin/
│   └── plugin.json
├── commands/
│   ├── deploy.md
│   └── status.md
├── skills/
│   └── code-review/
│       ├── SKILL.md
│       ├── scripts/
│       │   └── analyze.sh
│       └── references/
│           └── checklist.md
├── agents/
│   └── security-reviewer.md
├── rules/
│   └── prefer-const.mdc
├── hooks/
│   └── hooks.json
├── .mcp.json
├── .lsp.json
├── scripts/
│   ├── format.sh
│   └── check-env.sh
├── assets/
├── LICENSE
└── CHANGELOG.md
```

<!-- DISCUSSION: standard-layout-extensions — Should the standard layout include additional well-known directories such as `tests/` or `docs/`? -->

> **See also:** [§5 Manifest location and precedence](#5-manifest-location-and-precedence) for how the metadata directory relates to manifest discovery, and [§7 Component discovery](#7-component-discovery) for how component directories are scanned.

### 4.3 Directory rules

1. The metadata directory MUST contain `plugin.json`.
2. The metadata directory MAY also contain `marketplace.json` when the same directory root is both a plugin root and a marketplace root.
3. Component directories such as `skills/`, when present, MUST exist at the plugin root, not inside the metadata directory.
4. Missing component directories are not errors.

Example:

```text
my-plugin/
└── .plugin/
    ├── plugin.json
    └── marketplace.json
```

## 5. Manifest location and precedence

### 5.1 Manifest locations

Hosts MUST check for a manifest at `.plugin/plugin.json`.

A host that defines a vendor-prefixed manifest location such as `.<tool-name>-plugin/plugin.json` MUST also check that location and SHOULD prefer it over `.plugin/plugin.json` when both are present.

| Path                              | Description              | Notes                                   |
| --------------------------------- | ------------------------ | --------------------------------------- |
| `.plugin/plugin.json`             | Vendor-neutral manifest  | REQUIRED. RECOMMENDED for new multi-host plugins. |
| `.<tool-name>-plugin/plugin.json` | Vendor-specific manifest | Preferred by the matching host when present.      |

Example:

```text
my-plugin/
├── .plugin/plugin.json
└── .claude-plugin/plugin.json
```

A Claude-like host selects `.claude-plugin/plugin.json`. A host with no vendor-prefixed manifest location selects `.plugin/plugin.json`.

<!-- DISCUSSION: vendor-prefix-discovery — Should the spec define a registry or convention for vendor prefixes, or leave it entirely host-defined? -->

> **See also:** [§6 Manifest schema](#6-manifest-schema) for the structure of the manifest file, and [§12 Host conformance](#12-host-conformance) for requirements around supporting `.plugin/plugin.json`.

### 5.2 Multiple manifest locations

1. A plugin MAY provide identical manifest content in multiple locations.
2. When multiple manifest locations exist, a host SHOULD prefer its own vendor-prefixed manifest and SHOULD otherwise fall back to `.plugin/plugin.json`.
3. When both locations exist and contain different content, the selected manifest is authoritative for that host.
4. Hosts MAY warn when multiple manifest locations contain inconsistent content.

> **Implementer note:**
> Example user-facing message: `Plugin "devtools": manifest at ".claude-plugin/plugin.json" differs from ".plugin/plugin.json". Using ".claude-plugin/plugin.json" as authoritative.`
>
> Example machine-readable record:
> ```json
> {"level":"warn","event":"open_plugin.manifest.inconsistent","plugin":"devtools","selected":".claude-plugin/plugin.json","other":".plugin/plugin.json","action":"used_selected"}
> ```

## 6. Manifest schema

> **See also:** [§5 Manifest location and precedence](#5-manifest-location-and-precedence) for where the manifest is loaded from, and [§7 Component discovery](#7-component-discovery) for how manifest fields control discovery paths.

### 6.1 Manifest object

The manifest MUST be JSON and MUST contain a top-level object.

```json
{
  "name": "plugin-name",
  "version": "1.2.0",
  "description": "Brief plugin description",
  "author": {
    "name": "Author Name",
    "email": "author@example.com",
    "url": "https://example.com"
  },
  "homepage": "https://docs.example.com/plugin",
  "repository": "https://github.com/example/plugin",
  "license": "MIT",
  "keywords": ["keyword1", "keyword2"],
  "commands": ["./custom/commands/"],
  "agents": "./custom/agents/",
  "skills": "./custom/skills/",
  "rules": "./custom/rules/",
  "hooks": "./config/hooks.json",
  "mcpServers": "./mcp-config.json",
  "lspServers": "./.lsp.json",
  "outputStyles": "./styles/",
  "userConfig": {
    "api_key": {
      "description": "API key for the backend service",
      "sensitive": true
    },
    "output_format": {
      "description": "Preferred output format (json or text)"
    }
  }
}
```

Example: minimal manifest

```json
{
  "name": "minimal-plugin",
  "version": "1.0.0",
  "description": "The simplest possible plugin."
}
```

Example: full manifest

```json
{
  "name": "devtools",
  "version": "2.0.0",
  "description": "Full-featured development toolkit.",
  "author": {
    "name": "Open Plugin Examples"
  },
  "license": "Apache-2.0",
  "keywords": ["devtools", "code-review", "linting", "security"],
  "commands": "./commands/",
  "skills": {
    "paths": ["./skills/"]
  },
  "hooks": "./hooks/hooks.json",
  "mcpServers": "./.mcp.json",
  "lspServers": "./.lsp.json"
}
```

### 6.2 Required field

| Field  | Type   | Description                                                      |
| ------ | ------ | ---------------------------------------------------------------- |
| `name` | string | Unique plugin identifier used for namespacing and settings keys. |

### 6.3 Metadata fields

| Field         | Type     | Description                                                           |
| ------------- | -------- | --------------------------------------------------------------------- |
| `version`     | string   | Version string (Semantic Versioning RECOMMENDED). Used for update checks and cache freshness. |
| `description` | string   | Short description of plugin purpose.                                  |
| `author`      | object   | Author object with optional `name`, `email`, and `url` string fields. |
| `homepage`    | string   | Documentation or homepage URL.                                        |
| `repository`  | string   | Source repository URL.                                                |
| `license`     | string   | SPDX license identifier.                                              |
| `keywords`    | string[] | Search and discovery tags.                                            |

### 6.4 Component path fields

Core component path fields:

| Field          | Type                         | Description                                                |
| -------------- | ---------------------------- | ---------------------------------------------------------- |
| `skills`       | string \| string[] \| object | Skill directories, or a path config.                       |
| `mcpServers`   | string \| string[] \| object | MCP config paths, a path config, or inline MCP config.     |

Hosts MAY support additional component path fields for extended component types. See [Appendix E: Extended Component Types](#appendix-e-extended-component-types) for details.

When the manifest declares paths for a component type, those paths control discovery for that type. The default location is not scanned unless the manifest explicitly includes it.

Example: custom paths override the default

```json
{
  "skills": "./custom-skills/"
}
```

The host scans only `custom-skills/`; the default `skills/` directory is not scanned.

Example: retaining the default alongside custom paths

```json
{
  "skills": ["./skills/", "./extra-skills/"]
}
```

The host scans both `skills/` and `extra-skills/` because the default directory is explicitly included.

### 6.5 Path config schema

The object form for discovery paths is:

| Field   | Type     | Description                            |
| ------- | -------- | -------------------------------------- |
| `paths` | string[] | Relative files or directories to scan. |

### 6.6 Object-field disambiguation

Hosts MUST interpret object values for component path fields as follows:

| Field          | Object shape                                         | Interpretation      |
| -------------- | ---------------------------------------------------- | ------------------- |
| `mcpServers`   | Object containing `mcpServers`                       | Inline MCP config.  |
| Any path field | Object containing `paths`                            | Path config.        |

Hosts that support extended component types (see [Appendix E](#appendix-e-extended-component-types)) MAY define additional object shapes for those fields (e.g., inline hook config, inline LSP config).

If an object matches none of the shapes above, or matches more than one shape, the host SHOULD treat the field as invalid and SHOULD warn.

> **Implementer note:**
> Example user-facing message: `Plugin "devtools": manifest field "mcpServers" has an unrecognized shape (expected path config or inline server config). Field ignored; plugin load continues.`
>
> Example machine-readable record:
> ```json
> {"level":"warn","event":"open_plugin.manifest.invalid_object","plugin":"devtools","field":"mcpServers","action":"ignored","continue":true}
> ```

Example: path config

```json
{
  "skills": {
    "paths": ["./skills/", "./extra-skills/"]
  }
}
```

Example: inline MCP config

```json
{
  "mcpServers": {
    "mcpServers": {
      "database": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-postgres"]
      }
    }
  }
}
```

Example: invalid ambiguous object

```json
{
  "mcpServers": {
    "database": {
      "command": "npx"
    }
  }
}
```

This is invalid because it is neither a path config with `paths` nor an inline MCP config with a top-level `mcpServers` key.

### 6.7 Plugin name constraints

The manifest `name` value MUST satisfy all of the following:

| Constraint    | Requirement            | Description                                                   |
| ------------- | ---------------------- | ------------------------------------------------------------- |
| Length        | 1-64 characters        | The name MUST be between 1 and 64 characters inclusive.       |
| Character set | `a-z`, `0-9`, `-`, `.` | Lowercase alphanumeric characters, hyphens, and periods only. |
| Start and end | Alphanumeric           | The first and last characters MUST be alphanumeric.           |
| Repetition    | No `--` or `..`        | Consecutive hyphens and consecutive periods are not allowed.  |

Periods are allowed in plugin names.

Valid names: `my-plugin`, `acme.tools`, `lint3r`, `a`

Invalid names: `My-Plugin` (uppercase), `-start` (leading hyphen), `has--double` (consecutive hyphens), `too.many..dots` (consecutive periods), `` (empty)

### 6.8 User configuration

*User configuration is still under discussion and not required for v1 conformance. See [Appendix C: User Configuration](#appendix-c-user-configuration) for the current draft. Hosts MAY implement `userConfig` but conformant hosts are not required to.*

## 7. Component discovery

> **See also:** [§4 Plugin package model](#4-plugin-package-model) for directory layout conventions, [§6 Manifest schema](#6-manifest-schema) for how manifest fields declare discovery paths, and [§9 Namespacing](#9-namespacing) for how discovered components are named.

### 7.1 Default locations

Hosts MUST discover core components in the following default locations when no manifest field overrides discovery for that component type. When the manifest declares paths for a component type, those paths control discovery and the default location is not scanned unless explicitly included. See [§6.4](#64-component-path-fields) for details.

Core component locations:

| Component     | Default location   | Default pattern                      |
| ------------- | ------------------ | ------------------------------------ |
| Skills        | `skills/`          | Subdirectories containing `SKILL.md` |
| MCP servers   | `.mcp.json`        | JSON configuration                   |

Hosts MAY support extended component types with their own default locations. See [Appendix E: Extended Component Types](#appendix-e-extended-component-types) for details.

Example: given a plugin `reports-plugin` with this layout:

```text
reports-plugin/
├── .plugin/plugin.json
├── skills/summarize/SKILL.md
└── .mcp.json
```

The host discovers skill `summarize` and MCP servers from `.mcp.json` — all from default locations.

### 7.2 Missing locations

1. If a default location is absent, the host MUST NOT treat that as an error.
2. When the manifest declares paths for a component type, those paths control discovery. The default location is not scanned unless explicitly included.
3. When no manifest field is declared for a component type, the default location is scanned normally.

Example: a plugin declares `"skills": "./custom-skills/"` but has no `skills/` directory. The host scans only `custom-skills/` and does not error on the missing default `skills/` path.

### 7.3 Root `SKILL.md` fallback

If a plugin has no `skills/` directory and no manifest `skills` field, a host MAY treat `SKILL.md` at the plugin root as a single Agent Skill. In that case, the skill name MUST be derived from the plugin name.

> **Note:** Not all hosts implement the root `SKILL.md` fallback. Claude Code, for example, requires skills to be in `skills/` subdirectories. Plugin authors targeting multiple hosts SHOULD use the standard `skills/` directory layout.

Example:

```text
solo-skill/
├── .plugin/plugin.json
└── SKILL.md
```

If there is no `skills/` directory and no manifest `skills` field, a host MAY surface the root skill as `/solo-skill:solo-skill`.

### 7.4 Discovery examples

Example: default discovery only

```text
reports-plugin/
├── .plugin/plugin.json
├── skills/summarize/SKILL.md
└── .mcp.json
```

The host discovers skill `summarize` and MCP servers from `.mcp.json` — all from default locations.

Example: manifest paths override defaults

`./.plugin/plugin.json`

```json
{
  "name": "reports-plugin",
  "skills": "./custom-skills/"
}
```

```text
reports-plugin/
├── skills/summarize/SKILL.md
└── custom-skills/deploy/SKILL.md
```

The host discovers only `/reports-plugin:deploy` from `custom-skills/`. The default `skills/summarize/` is not discovered because the manifest declares custom paths.

Example: retaining the default alongside custom paths

`./.plugin/plugin.json`

```json
{
  "name": "reports-plugin",
  "skills": ["./skills/", "./custom-skills/"]
}
```

The host discovers both `/reports-plugin:summarize` and `/reports-plugin:deploy` because the default directory is explicitly included.

## 8. Component definitions

> **See also:** [§7 Component discovery](#7-component-discovery) for how component files are located, and [§9 Namespacing](#9-namespacing) for how component names are surfaced to users and models.

This specification normatively defines discovery for two component types that are backed by open standards: **skills** and **MCP servers**. Hosts MAY support additional component types (commands, agents, rules, hooks, LSP servers, output styles). See [Appendix E: Extended Component Types](#appendix-e-extended-component-types) for reference definitions of these additional types.

Hosts MUST ignore component types they do not support.

### 8.1 Skills

Agent Skills follow the [Agent Skills specification](https://agentskills.io/specification). The Agent Skills spec is the source of truth for the `SKILL.md` format, frontmatter fields, and directory layout (`scripts/`, `references/`, `assets/`).

This specification defines how Agent Skills are *discovered* and *namespaced* within a plugin, not the skill format itself.

The default discovery location is `skills/`. Each subdirectory containing `SKILL.md` is treated as one skill.

Example: a skill directory named `deploy` inside `skills/`:

```text
skills/
  deploy/
    SKILL.md          # name: deploy
    scripts/
      rollback.sh
    references/
      runbook.md
```

### 8.2 MCP servers

MCP server configuration follows the [Model Context Protocol specification](https://modelcontextprotocol.io/specification). The MCP spec is the source of truth for server configuration fields, transport types (stdio, HTTP/SSE), and lifecycle semantics.

This specification defines how MCP servers are *discovered* within a plugin and how `${PLUGIN_ROOT}` expansion applies to configuration values.

#### 8.2.1 Discovery and configuration

The default MCP configuration path is `.mcp.json`. MCP servers MAY also be declared inline in the manifest `mcpServers` field.

The configuration MUST contain a top-level `mcpServers` object. Each key is the server name and MUST be unique within the plugin after source resolution.

The `command`, `args`, `env`, and `cwd` fields in MCP server configuration MUST support `${PLUGIN_ROOT}` expansion.

Example: `.mcp.json`

```json
{
  "mcpServers": {
    "database": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_URL": "postgresql://localhost:5432/mydb"
      }
    },
    "filesystem": {
      "command": "${PLUGIN_ROOT}/bin/fs-server",
      "args": ["--root", "${PLUGIN_ROOT}/data"],
      "cwd": "${PLUGIN_ROOT}"
    }
  }
}
```

#### 8.2.2 Plugin-specific discovery rules

1. When no `mcpServers` manifest field is declared, hosts MUST discover MCP configuration from the default `.mcp.json` location.
2. When the manifest declares `mcpServers` paths, those paths MUST point to explicit JSON files (not directories). Each file MUST contain a top-level `mcpServers` object. When the manifest declares inline MCP config, that config is used directly.
3. Manifest-declared paths control discovery per the rules in [§6.4](#64-component-path-fields).
4. If a server fails to start, the host SHOULD log the error and continue loading other components.
5. If multiple discovery sources define the same MCP server name, behavior is implementation-defined. Hosts SHOULD warn and SHOULD resolve conflicts deterministically. Conflicting MCP server names across discovery sources MUST NOT crash plugin loading.

## 9. Namespacing

<!-- DISCUSSION: namespacing-separator — The colon separator `plugin:component` was chosen over `/` and `__` for general components. Should MCP tool namespacing also use colons, or does underscore-based `mcp__plugin_...` remain necessary for model compatibility? -->

> **See also:** [§6.7 Plugin name constraints](#67-plugin-name-constraints) for allowed characters in plugin names, and [§8 Component definitions](#8-component-definitions) for the naming rules of each component type.

### 9.1 General component namespacing

Hosts SHOULD namespace plugin-provided components to avoid collisions. The RECOMMENDED format is:

```text
{plugin-name}:{component-name}
```

Example:

```text
deploy-tools:status
```

### 9.2 MCP tool identifier namespacing

When surfacing MCP tools to a model, hosts SHOULD include both the plugin name and the server name to avoid collisions.

The RECOMMENDED surfaced identifier format is:

```text
mcp__plugin_{plugin-name}_{server-name}__{tool-name}
```

Example:

| Plugin         | Server     | Tool    | Identifier                                 |
| -------------- | ---------- | ------- | ------------------------------------------ |
| `deploy-tools` | `database` | `query` | `mcp__plugin_deploy-tools_database__query` |

Hosts that use a different naming convention MAY adapt this format.

Example: component namespacing

| Source file | Component name | Surfaced identifier |
| --- | --- | --- |
| `skills/code-review/SKILL.md` | `code-review` | `devtools:code-review` |
| `skills/deploy/SKILL.md` | `deploy` | `devtools:deploy` |

Example: MCP tool namespacing

| Plugin | Server | Tool | Surfaced identifier |
| --- | --- | --- | --- |
| `devtools` | `database` | `query` | `mcp__plugin_devtools_database__query` |
| `devtools` | `database` | `migrate` | `mcp__plugin_devtools_database__migrate` |
| `deploy-tools` | `kubernetes` | `rollout_status` | `mcp__plugin_deploy-tools_kubernetes__rollout_status` |

## 10. Environment variables and placeholder expansion

> **See also:** [§8.2 MCP servers](#82-mcp-servers) for the fields where `${PLUGIN_ROOT}` expansion applies, and [§4.1 General requirements](#41-general-requirements) for path safety rules.

### 10.1 Required variables

Hosts that launch plugin subprocesses (e.g., MCP servers, hook commands) MUST provide an environment variable containing the absolute plugin root path.

| Variable      | Description             | Notes                                                      |
| ------------- | ----------------------- | ---------------------------------------------------------- |
| `PLUGIN_ROOT` | Absolute plugin root path | REQUIRED for hosts that launch plugin subprocesses. |

### 10.1.1 Persistent data directory

Hosts that provide persistent plugin storage SHOULD set a data directory variable.

| Variable      | Description                  | Notes        |
| ------------- | ---------------------------- | ------------ |
| `PLUGIN_DATA` | Persistent data directory path | RECOMMENDED. |

`PLUGIN_DATA` is the absolute path to a host-managed persistent data directory for the plugin. This directory SHOULD survive plugin updates and reinstalls. The host SHOULD create the directory on first reference and MAY delete it when the plugin is uninstalled.

Use `PLUGIN_DATA` for: installed dependencies (node_modules, virtual environments), generated code, caches, and other plugin state that should persist across updates. Use `PLUGIN_ROOT` for referencing bundled scripts, binaries, and config files that ship with the plugin.

Example: a host loading the plugin `devtools` from `/home/alex/.agents/plugins/devtools` sets:

```text
PLUGIN_ROOT=/home/alex/.agents/plugins/devtools
PLUGIN_DATA=/home/alex/.agents/plugins/data/devtools
```

Hosts MAY additionally provide vendor-prefixed equivalents (e.g., `CLAUDE_PLUGIN_ROOT`) but these are host-specific and not defined by this spec.

### 10.2 Placeholder expansion

Hosts that provide `PLUGIN_ROOT` MUST expand `${PLUGIN_ROOT}` in supported configuration fields. Hosts that provide a persistent data directory SHOULD expand `${PLUGIN_DATA}`.

Generic environment variable interpolation (e.g., `${HOME}`) is not required by this spec. Hosts MAY support it.

Expansion applies to runtime configuration values — executable paths and environment-bearing strings — not to manifest discovery path fields. Manifest discovery paths (§6.4) are always relative `./` paths resolved from the plugin root.

| Location                     | Fields                                                                                       | Description                                            |
| ---------------------------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| MCP config                   | `command`, `args`, `env`, `cwd`                                                              | All string values support plugin-root expansion.       |

Hosts MAY provide additional environment variables beyond the plugin root variables.

Example: `${PLUGIN_ROOT}` expansion in MCP

```json
{
  "mcpServers": {
    "database": {
      "command": "npx",
      "args": ["--config", "${PLUGIN_ROOT}/config/db.json"],
      "cwd": "${PLUGIN_ROOT}",
      "env": {
        "DATA_DIR": "${PLUGIN_ROOT}/data"
      }
    }
  }
}
```

## 11. Versioning

Plugins SHOULD use Semantic Versioning for `version`.

| Segment | Meaning                     | Description                                            |
| ------- | --------------------------- | ------------------------------------------------------ |
| Major   | Breaking change             | Incompatible behavior or schema change.                |
| Minor   | Backward-compatible feature | New behavior without breaking existing hosts or users. |
| Patch   | Backward-compatible fix     | Corrective change without intended behavioral break.   |

Hosts MAY use `version` to determine whether updates are available and whether caches are stale.

## 12. Host conformance

### 12.1 Minimum host requirements

A host is conformant to Open Plugin v1 if it:

1. Can load a plugin from a directory path.
2. Parses `.plugin/plugin.json`.
3. For each core component type it supports (skills, MCP servers), discovers components in default locations.
4. Respects manifest-declared discovery paths for supported component types.
5. If the host launches plugin subprocesses (e.g., MCP servers), expands `${PLUGIN_ROOT}` in runtime configuration values (`command`, `args`, `env`, `cwd`).
6. Supports at least one core component type (skills or MCP servers).

Support for a vendor-prefixed manifest location such as `.<tool-name>-plugin/plugin.json` is supplemental. A host that defines such a location MUST still support `.plugin/plugin.json`.

Example: a skills-only host is conformant. It only needs to:

```text
1. Accept a plugin directory path.
2. Read .plugin/plugin.json for the plugin name.
3. Scan skills/ for SKILL.md files (default location discovery).
4. Respect manifest-declared skill paths.
```

A host that only supports skills — and ignores MCP servers, commands, agents, rules, hooks, LSP servers, and output styles — is fully conformant to Open Plugin v1 as long as it meets all six requirements above.

Support for extended component types (commands, agents, rules, hooks, LSP servers, output styles) is OPTIONAL. See [Appendix E: Extended Component Types](#appendix-e-extended-component-types).

### 12.2 Incremental adoption

A host is not required to support every component type. Incremental adoption is conformant.

### 12.3 Unsupported features

1. Hosts MUST ignore unsupported component types.
2. Hosts MUST continue loading a plugin when a component fails independently, such as an MCP server startup failure, unless the host explicitly treats that failure as fatal for that component only.
3. Hosts SHOULD warn when configuration is invalid, conflicting, or partially unsupported.

---

## Appendix A: Conformance Checklist

*This checklist is for convenience only — when it conflicts with the spec text above, the spec governs.*

### Plugin loader

- [ ] Parse `.plugin/plugin.json` ([§5.1](#51-manifest-locations))
- [ ] Support vendor-prefixed manifest locations if applicable ([§5.1](#51-manifest-locations))
- [ ] Validate plugin name against naming constraints ([§6.7](#67-plugin-name-constraints))
- [ ] Reject paths that escape the plugin root via `../` ([§4.1](#41-general-requirements))

### Component discovery

- [ ] Scan default locations for each supported component type ([§7.1](#71-default-locations))
- [ ] Respect manifest-declared discovery paths ([§6.4](#64-component-path-fields), [§7.2](#72-missing-locations))
- [ ] Ignore missing default directories without error ([§7.2](#72-missing-locations))

### Namespacing

- [ ] SHOULD namespace components as `{plugin-name}:{component-name}` ([§9.1](#91-general-component-namespacing))
- [ ] SHOULD include both plugin name and server name in MCP tool identifiers ([§9.2](#92-mcp-tool-identifier-namespacing))

### Environment and expansion

- [ ] If the host launches plugin subprocesses, provide `PLUGIN_ROOT` environment variable ([§10.1](#101-required-variables))
- [ ] If the host provides `PLUGIN_ROOT`, expand `${PLUGIN_ROOT}` in MCP server `command`, `args`, `env`, `cwd` fields ([§10.2](#102-placeholder-expansion))

### Resilience

- [ ] Ignore unsupported component types ([§12.3](#123-unsupported-features))
- [ ] Continue loading when optional components fail ([§12.3](#123-unsupported-features))
- [ ] Support at least one core component type ([§12.1](#121-minimum-host-requirements))

### Diagnostics matrix

*This table consolidates the failure-handling behaviors defined throughout the spec into a single reference. All behaviors listed below restate existing requirements — no new requirements are introduced. Host implementers can use this matrix to verify that their diagnostic output covers every specified failure site.*

<!-- DISCUSSION: diagnostics-contract -->

| Decision point | Existing spec behavior | Human-readable example | JSON example | Fatal? | Verification |
| --- | --- | --- | --- | --- | --- |
| Inconsistent manifest content | Hosts MAY warn when multiple manifest locations contain inconsistent content ([§5.2](#52-multiple-manifest-locations)) | `WARN open-plugin: plugin "devtools" has inconsistent manifests across .plugin/plugin.json and .claude-plugin/plugin.json; using .claude-plugin/plugin.json` | `{"level":"warn","event":"open_plugin.manifest.inconsistent","plugin":"devtools","selected":".claude-plugin/plugin.json","other":".plugin/plugin.json","action":"used_selected"}` | No | Check that the host selects one manifest and continues |
| Invalid ambiguous object field | Hosts SHOULD treat unrecognized object shapes as invalid and SHOULD warn ([§6.6](#66-object-field-disambiguation)) | `WARN open-plugin: plugin "devtools" manifest field "mcpServers" is invalid: expected either a path config with "paths" key or an inline config with "mcpServers" key; field ignored, plugin load continues` | `{"level":"warn","event":"open_plugin.manifest.invalid_object","plugin":"devtools","field":"mcpServers","action":"ignored","continue":true}` | No | Check that the invalid field is skipped and remaining components load |
| MCP server startup failure | If a server fails to start, the host SHOULD log the error and continue loading other components ([§8.2.2](#822-plugin-specific-discovery-rules)) | `ERROR open-plugin: plugin "devtools" MCP server "database" failed to start: connection refused on port 5432. Other plugin components remain available.` | `{"level":"error","event":"open_plugin.mcp.start_failed","plugin":"devtools","server":"database","error":"connection refused on port 5432","action":"continue_without_mcp"}` | No | Check that other components still load |
| MCP server name conflict | Hosts SHOULD warn and SHOULD resolve conflicts deterministically. Conflicting names MUST NOT crash plugin loading ([§8.2.2](#822-plugin-specific-discovery-rules)) | `WARN open-plugin: plugin "devtools" MCP server name "filesystem" defined in multiple config files; using first definition` | `{"level":"warn","event":"open_plugin.mcp.name_conflict","plugin":"devtools","server":"filesystem","action":"used_first"}` | No | Check that one definition wins deterministically and loading continues |
| Unsupported component type | Hosts MUST ignore unsupported component types ([§12.3](#123-unsupported-features)) | `INFO open-plugin: plugin "devtools" declares component type "agents" which is not supported by this host; ignored` | `{"level":"info","event":"open_plugin.host.unsupported_component","plugin":"devtools","component_type":"agents","action":"ignored"}` | No | Check that supported components still load |
| Partial host support | Hosts SHOULD warn when configuration is invalid, conflicting, or partially unsupported ([§12.3](#123-unsupported-features)) | `WARN open-plugin: plugin "devtools" is partially supported: this host supports skills and hooks but not mcpServers or lspServers` | `{"level":"warn","event":"open_plugin.host.partial_support","plugin":"devtools","supported":["skills","hooks"],"unsupported":["mcpServers","lspServers"],"action":"loaded_partial"}` | No | Check that supported components are functional |

> **Implementer note:** The `event` field values above (e.g., `open_plugin.manifest.invalid_object`) are *suggested* stable identifiers — not required by this spec. Hosts that adopt them gain a machine-readable diagnostic surface that agents, CI pipelines, and plugin validators can consume deterministically. The recommended fields for every diagnostic record are: `level`, `event`, `plugin` (plugin name), the relevant component identifier (e.g., `server`, `field`, `hook_event`), and `action` (what the host did in response).

---

## Design Decisions

*This section explains why key design choices were made. It is for context only — the binding rules are in sections 1–12 above.*

### Why directory-based discovery?

Plugins use filesystem directories as the package unit rather than archive formats (`.zip`, `.tar.gz`) or registry-fetched bundles. This keeps plugins inspectable with standard tools (`ls`, `cat`, `git`), editable in-place during development, and compatible with version control without special tooling.

### Why colon-separated namespacing for components?

The `plugin-name:component-name` format was chosen because colons are visually distinct, rarely appear in filenames, and align with existing conventions in tools like Claude Code's slash commands (`/plugin:command`). Alternatives considered included `/` (conflicts with filesystem paths) and `__` (less readable for user-facing identifiers).

### Why underscore-based namespacing for MCP tools?

MCP tool identifiers are consumed by language models, which may tokenize or interpret colons and slashes unpredictably. The `mcp__plugin_{plugin}_{server}__{tool}` format uses only characters that models handle reliably. The double-underscore separators provide unambiguous parsing boundaries even when plugin or tool names contain single underscores.

### Why `.plugin/plugin.json` is the conformance floor

Every conformant host MUST check `.plugin/plugin.json` as the vendor-neutral manifest location ([§5.1](#51-manifest-locations)). This gives plugin authors a single guaranteed path that works across all hosts without vendor-specific knowledge. Vendor-prefixed manifests (e.g., `.claude-plugin/plugin.json`) are supplemental overrides — they let a plugin customize behavior for a specific host, but a plugin that ships only `.plugin/plugin.json` is portable by default. Making the vendor-neutral path mandatory and vendor-prefixed paths optional keeps the ecosystem interoperable while allowing per-host specialization where needed.

### Why `.plugin/` instead of a dotfile manifest?

A metadata directory (`.plugin/`) allows the manifest to coexist with future metadata files (e.g., `marketplace.json`, lock files) without polluting the plugin root with multiple dotfiles. A single well-known directory is also easier for hosts to check than scanning for multiple dotfile patterns.

### Why `${PLUGIN_ROOT}` over relative paths in configs?

Hook commands, MCP server arguments, and LSP configurations often need absolute paths at runtime. Relative paths from the config file location would be ambiguous when configs are loaded from different directories. `${PLUGIN_ROOT}` provides an unambiguous, host-resolved anchor that works regardless of the current working directory.

### Why optional-component failures are non-fatal

When an MCP server fails to start, an LSP binary is missing, or a hook command exits non-zero, the host continues loading the remaining components ([§12.3](#123-unsupported-features)). This design reflects the reality that plugins bundle heterogeneous components with different runtime dependencies — a plugin that provides skills, hooks, *and* an MCP server should not become entirely unusable because one server's port is occupied. Non-fatal failures also make plugins more resilient in constrained environments (CI runners, containers, minimal installs) where not every runtime dependency is available. The spec pairs this with diagnostic requirements so that failures are visible rather than silent.

<!-- DISCUSSION: diagnostics-contract -->

---

## Future Considerations

*Everything in this section is deferred from v1.0.0. Nothing here is required for conformance. These items may be addressed in future versions.*

### Permission and approval UX

v1.0.0 does not define a trust model, permission system, or sandboxing requirements for plugins. A future version should address:

- Permission declarations in the manifest (e.g., filesystem access, network access, tool access)
- Host-enforced capability restrictions per plugin
- User consent flows for plugin installation and capability grants
- Approval UX for hooks and MCP servers that execute arbitrary commands or access external services
- Graduated trust levels (e.g., "sandboxed", "user-approved", "organization-approved")

### Provenance verification

v1.0.0 does not specify how hosts or users can verify the origin or integrity of a plugin. A future version may define:

- Cryptographic signature verification for published plugins
- Attestation chains linking a published plugin to its source repository and build
- Host-side policies for requiring signatures from trusted publishers

### Secret and sensitive value handling

Plugin components (hooks, MCP servers, LSP servers) often need credentials or API keys at runtime. v1.0.0 does not specify how sensitive values should be provided, stored, or scoped. A future version may define:

- A `secrets` manifest field or separate secrets configuration
- Host-mediated secret injection that avoids plaintext in config files
- Scoping rules that prevent one plugin from accessing another plugin's secrets
- Rotation and revocation semantics for plugin-held credentials

### Enterprise controls

Organizations deploying plugins at scale need policy enforcement that v1.0.0 does not address. A future version may define:

- Allowlist and blocklist policies for plugin installation by name, publisher, or signature
- Organization-scoped plugin registries with approval workflows
- Centralized configuration overrides that take precedence over user-level plugin settings
- Compliance reporting hooks for plugin installation and usage events

### Audit-trail standardization

v1.0.0 defines diagnostic events for failure sites but does not standardize lifecycle audit events. A future version may define:

- A standard event schema for plugin install, enable, disable, update, and uninstall actions
- Recommended fields: timestamp, actor (user or automation), plugin name, plugin version, action, outcome
- Integration points for forwarding audit events to external logging or SIEM systems
- Retention and access policies for audit records

### Dependency resolution

Plugins currently cannot declare dependencies on other plugins. A future version may define:

- A `dependencies` manifest field with version constraints
- Resolution order and conflict handling for transitive dependencies
- Peer dependency semantics for shared components

### Binary distribution

v1.0.0 assumes plugins are source-distributable (scripts, markdown, JSON configs). A future version may address:

- Precompiled binary distribution for MCP and LSP servers
- Platform-specific binary selection
- Integrity verification for binary artifacts

### Message channels

Some hosts support a `channels` manifest field for message channel injection (e.g., Telegram, Slack, Discord style integrations) bound to MCP servers. This feature is host-specific and too specialized for v1 core. A future version may define:

- A `channels` manifest field with per-channel MCP server bindings
- Per-channel user configuration for credentials and settings
- A standard protocol for channel message injection into conversations

### Output styles runtime semantics

The `outputStyles` field is defined only at the discovery level. A future version should specify the runtime format and rendering contract for output style resources.

### Plugin testing and validation

No test harness or validation tool is specified. A future version may define:

- A `test` manifest field or convention
- A standard plugin linter or validator command
- Conformance test suites for host implementations

---

## Appendix B: Marketplace Index and Discovery

*This appendix is not required for v1 conformance. It describes a marketplace indexing mechanism implemented by some hosts. A future version of the spec may promote this to a required section after cross-tool validation.*

### B.1 Marketplace discovery order

Hosts that support marketplaces should search for `marketplace.json` in this order:

| Path                                   | Priority | Description                         |
| -------------------------------------- | -------- | ----------------------------------- |
| `marketplace.json`                     | First    | Marketplace root index.             |
| `.plugin/marketplace.json`             | Second   | Vendor-neutral metadata directory.  |
| `.<tool-name>-plugin/marketplace.json` | Third    | Vendor-specific metadata directory. |

The first match wins.

### B.2 Marketplace schema

Required fields:

| Field     | Type   | Description                        |
| --------- | ------ | ---------------------------------- |
| `name`    | string | Marketplace identifier.            |
| `plugins` | array  | Non-empty array of plugin entries. |

Optional fields:

| Field                 | Type   | Description                                                   |
| --------------------- | ------ | ------------------------------------------------------------- |
| `owner`               | object | Marketplace owner with `name` and optional `email` and `url`. |
| `metadata`            | object | Marketplace-level metadata.                                   |
| `metadata.pluginRoot` | string | Base path for plugin `source` resolution. Defaults to `.`.    |

### B.3 Plugin entries

| Field         | Type     | Description                                                                                  |
| ------------- | -------- | -------------------------------------------------------------------------------------------- |
| `name`        | string   | Required plugin name. Must satisfy plugin name constraints.                                  |
| `source`      | string   | Required relative path from `metadata.pluginRoot` or marketplace root. Must start with `./`. |
| `description` | string   | Optional marketplace description override.                                                   |
| `version`     | string   | Optional marketplace version override for update checks.                                     |
| `author`      | object   | Optional marketplace author override.                                                        |
| `license`     | string   | Optional marketplace license override.                                                       |
| `keywords`    | string[] | Optional marketplace keywords override.                                                      |
| `skills`      | string[] | Optional explicit skill paths relative to the marketplace root.                              |

For marketplace-level operations such as display, search, and update checks, plugin-entry metadata overrides manifest metadata. Runtime plugin behavior continues to use the plugin manifest and plugin contents.

Example:

```json
{
  "name": "acme-plugins",
  "owner": { "name": "Acme Corp", "url": "https://acme.example.com" },
  "metadata": { "pluginRoot": "./plugins" },
  "plugins": [
    {
      "name": "code-review",
      "source": "./code-review",
      "description": "Automated code review with security checks.",
      "version": "2.1.0",
      "keywords": ["review", "security"]
    },
    {
      "name": "deploy-tools",
      "source": "./deploy-tools",
      "version": "1.0.3",
      "license": "Apache-2.0"
    }
  ]
}
```

### B.4 `pluginRoot` resolution

`metadata.pluginRoot` defines the base directory for resolving each plugin entry `source`. If omitted, `source` is resolved relative to the directory containing `marketplace.json`.

### B.5 Fallback scanning without an index

If no marketplace index is found, hosts should check whether the root directory itself is a plugin, scan immediate subdirectories, and scan one additional level deep.

### B.6 Marketplace name derivation

| Source                        | Derived name                         |
| ----------------------------- | ------------------------------------ |
| GitHub shorthand `owner/repo` | `owner-repo`                         |
| Git URL                       | Last two path segments joined by `-` |
| Local directory path          | Directory basename                   |

---

## Appendix C: User Configuration

*This appendix is not required for v1 conformance. It describes a user configuration mechanism implemented by some hosts. Hosts MAY implement `userConfig`. A future version of the spec may promote this to a required section.*

### C.1 Schema

The optional `userConfig` manifest field allows plugins to declare user-configurable values that the host prompts for when the plugin is enabled.

Each key in `userConfig` maps to a config descriptor:

| Field         | Type    | Description                                                                                       |
| ------------- | ------- | ------------------------------------------------------------------------------------------------- |
| `description` | string  | Required human-readable description of the configuration value.                                   |
| `sensitive`   | boolean | If `true`, the host should store this value securely (e.g., system keychain). Defaults to `false`. |

Configuration keys must be valid identifiers (alphanumeric characters and underscores).

Example:

```json
{
  "userConfig": {
    "api_key": {
      "description": "API key for the backend service",
      "sensitive": true
    },
    "output_format": {
      "description": "Preferred output format (json or text)"
    }
  }
}
```

### C.2 Value substitution

User config values are available as `${user_config.KEY}` placeholders in MCP/LSP server configurations, hook commands, and (for non-sensitive values only) skill and agent content.

### C.3 Environment variable export

Hosts should export user config values to plugin subprocesses as environment variables using the pattern `PLUGIN_OPTION_<KEY>`. Hosts may additionally export vendor-prefixed equivalents such as `CLAUDE_PLUGIN_OPTION_<KEY>`.

### C.4 Storage

Non-sensitive values should be stored in the host's settings file. Sensitive values should be stored in a secure credential store (e.g., system keychain).

---

## Appendix D: Extended Hook Events

*This appendix is not required for v1 conformance. It catalogs hook events implemented by existing hosts. Hosts MAY support any subset of these. Plugin authors should check host documentation for supported events. See [Appendix E.4](#e4-hooks) for the hook format definition.*

| Event                | Matcher context   | Description                                       | Known hosts  |
| -------------------- | ----------------- | ------------------------------------------------- | ------------ |
| `UserPromptSubmit`   | None              | Fires when a user prompt is submitted.            | Claude Code  |
| `Stop`               | None              | Fires when the agent finishes responding.         | Claude Code  |
| `StopFailure`        | Error type        | Fires when a turn ends due to an API error.       | Claude Code  |
| `SubagentStart`      | Agent type        | Fires when a sub-agent starts.                    | Claude Code  |
| `SubagentStop`       | Agent type        | Fires when a sub-agent stops.                     | Claude Code  |
| `PreCompact`         | Trigger           | Fires before context compaction.                  | Claude Code  |
| `PostCompact`        | Trigger           | Fires after context compaction completes.         | Claude Code  |
| `TeammateIdle`       | None              | Fires when a teammate agent is about to idle.     | Claude Code  |
| `TaskCreated`        | None              | Fires when a task is created.                     | Claude Code  |
| `TaskCompleted`      | None              | Fires when a task is marked completed.            | Claude Code  |
| `Notification`       | Notification type | Fires when the host sends a notification.         | Claude Code  |
| `PermissionRequest`  | None              | Fires when a permission dialog is shown.          | Claude Code  |
| `InstructionsLoaded` | Load reason       | Fires when instruction files are loaded.          | Claude Code  |
| `ConfigChange`       | Config source     | Fires when a configuration file changes.          | Claude Code  |
| `CwdChanged`         | None              | Fires when the working directory changes.         | Claude Code  |
| `FileChanged`        | Filename          | Fires when a watched file changes on disk.        | Claude Code  |
| `WorktreeCreate`     | None              | Fires when a worktree is being created.           | Claude Code  |
| `WorktreeRemove`     | None              | Fires when a worktree is being removed.           | Claude Code  |
| `Elicitation`        | MCP server name   | Fires when an MCP server requests user input.     | Claude Code  |
| `ElicitationResult`  | MCP server name   | Fires when a user responds to an MCP elicitation. | Claude Code  |

As additional hosts adopt the Open Plugin format, this table will be updated with cross-host event support information. Events supported by multiple hosts are candidates for promotion to the core set in future spec versions.

---

## Appendix E: Extended Component Types

*This appendix is not required for v1 conformance. It defines component types that hosts MAY support beyond the core skill and MCP server types. These formats are based on conventions established by existing hosts but are not yet backed by independent open standards. A future version of the spec may promote some of these to core component types.*

### E.1 Command skills

Command skills are markdown files discovered from `commands/`.

1. The filename without extension is the command name.
2. Command files MAY contain YAML frontmatter.

| Field                      | Type    | Description                                                              |
| -------------------------- | ------- | ------------------------------------------------------------------------ |
| `description`              | string  | Short description for display and matching.                              |
| `disable-model-invocation` | boolean | If `true`, require explicit user invocation. Defaults to `false`.        |

The markdown body is the command instruction body. The placeholder `$ARGUMENTS` in a command body is replaced with user-provided text after the command name.

### E.2 Agents

Agent definitions are markdown files with YAML frontmatter discovered from `agents/`.

| Field         | Type   | Description                                                  |
| ------------- | ------ | ------------------------------------------------------------ |
| `name`        | string | Required agent identifier (1-64 chars, lowercase alphanumeric and hyphens). |
| `description` | string | Required description (max 1024 characters).                  |

The markdown body after frontmatter is the agent system prompt.

### E.3 Rules

Rule files are markdown with YAML frontmatter discovered from `rules/`. The default extension is `.mdc`.

| Field         | Type               | Description                                                       |
| ------------- | ------------------ | ----------------------------------------------------------------- |
| `description` | string             | Required summary of the rule.                                     |
| `alwaysApply` | boolean            | If `true`, the rule is active automatically. Defaults to `false`. |
| `globs`       | string \| string[] | Optional file glob or glob list limiting rule applicability.      |

The markdown body is the rule text injected when the rule is active.

### E.4 Hooks

The default hook configuration path is `hooks/hooks.json`. Hooks MAY also be declared inline in the manifest `hooks` field.

The hook configuration contains a top-level `hooks` object. Each event key maps to an array of hook rules.

| Field     | Type   | Description                                                         |
| --------- | ------ | ------------------------------------------------------------------- |
| `matcher` | string | Optional regular expression matched against event-specific context. |
| `hooks`   | array  | Required list of hook actions.                                      |

Core hook events: `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `SessionStart`, `SessionEnd`. See [Appendix D: Extended Hook Events](#appendix-d-extended-hook-events) for additional events.

Hook action types: `command` (shell command), `http` (POST to URL), `prompt` (LLM evaluation), `agent` (agentic verifier).

### E.5 LSP servers

The default LSP configuration path is `.lsp.json`. LSP servers MAY also be declared inline in the manifest `lspServers` field. The top-level object uses direct server-name keys.

Required fields: `command` (executable on `$PATH`), `extensionToLanguage` (file extension to language ID mapping).

Optional fields: `args`, `transport`, `env`, `initializationOptions`, `settings`, `workspaceFolder`, `startupTimeout`, `shutdownTimeout`, `restartOnCrash`, `maxRestarts`.

### E.6 Output styles

`outputStyles` is a discovery field for host-defined output style resources. This specification does not define output style runtime semantics beyond discovery.
