# Open Plugin Specification

**Spec Version: 1.0.0**

This document defines the canonical Open Plugin Specification v1.0.0. It is a self-contained specification for packaging agent extensions into distributable plugins. Everything in sections 1–12 is required for conformance.

## Quick Start (for context only — not required for conformance)

The smallest useful plugin is a directory with one skill.

```text
hello-plugin/
├── plugin.json
└── skills/
    └── greet/
        └── SKILL.md
```

`./plugin.json`

```json
{
  "id": "https://github.com/example/hello-plugin/tree/main",
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

A host that supports skills can load this plugin by reading `plugin.json`, discovering `skills/greet/SKILL.md`, and surfacing `/hello-plugin:greet`.

> **Note:**
> Open Plugin v1 standardizes two component types: Agent Skills and MCP servers. Other capabilities are outside the portable v1 format.

## Table of contents

1. [Status and version](#1-status-and-version)
2. [Conformance language](#2-conformance-language)
3. [Terminology](#3-terminology)
4. [Plugin package model](#4-plugin-package-model)
5. [Manifest location and extension fields](#5-manifest-location-and-extension-fields)
6. [Manifest schema](#6-manifest-schema)
7. [Component discovery](#7-component-discovery)
8. [Component definitions](#8-component-definitions)
9. [Namespacing](#9-namespacing)
10. [Environment variables and placeholder expansion](#10-environment-variables-and-placeholder-expansion)
11. [Versioning](#11-versioning)
12. [Host conformance](#12-host-conformance)

**Appendices (not required for conformance)**

- [Appendix A: Conformance Checklist](#appendix-a-conformance-checklist)
- [Design Decisions](#design-decisions)
- [Future Considerations](#future-considerations)

## 1. Status and version

This specification defines version `1.0.0` of the Open Plugin format.

Plugin hosts and plugin packages claiming conformance to Open Plugin v1 MUST implement or follow the requirements in this document.

### 1.1 Governance model

Open Plugin is intended to be governed as a collaborative open standard, not as a single-vendor format. The initial governance model should be defined before the first ratified release and SHOULD use the Open Responses governance structure as a starting point.

The governance process should define:

1. A steering committee with one primary representative per participating implementation or vendor.
2. A lightweight quorum and voting process for accepting changes to the core specification.
3. A process for promoting widely adopted vendor extensions into the core specification.
4. A shared collaboration space for implementers, initially kept small enough to move quickly.
5. Neutral stewardship of public assets such as `openplugins.org` once the committee is formed.

Vendor-specific experimentation is expected. The committee's role is to identify extensions that have proven useful across implementations and decide when they should become portable core behavior.

The Project's governance rules are defined in the [Technical Charter](./GOVERNANCE.md).

## 2. Conformance language

The key words MUST, MUST NOT, REQUIRED, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL in sections 1–12 of this document are to be interpreted as described in RFC 2119 and RFC 8174 when, and only when, they appear in all capitals. Appendices and other non-normative sections use these terms informally.

## 3. Terminology

| Term               | Meaning                    | Description                                                                                                                                  |
| ------------------ | -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| Plugin             | Package unit               | A self-contained directory that bundles one or more components and optional metadata.                                                        |
| Plugin root        | Filesystem root            | The top-level directory of a plugin package.                                                                                                 |
| Manifest           | Metadata document          | A `plugin.json` file at the plugin root.                                                                                                    |
| Component          | Plugin-provided capability | A skill or MCP server configuration.                                                                                                         |
| Host               | Plugin runtime             | A tool that discovers, installs, loads, and executes plugin components.                                                                      |
| Discovery source   | Scan location              | A default location, manifest-declared path, or inline configuration object from which a host loads components.                               |
| Path config        | Discovery object           | An object with `paths` that controls scanning for a component type.                                                                          |
| Inline MCP config  | MCP definition object      | A `mcpServers` manifest field whose object value contains an `mcpServers` key.                                                               |
| Vendor extension   | Namespaced field           | A top-level manifest object reserved for one implementation, vendor, or host family.                                                         |

## 4. Plugin package model

### 4.1 General requirements

1. A plugin is a directory rooted at a single filesystem location.
2. A plugin MUST include a manifest at `plugin.json` in the plugin root.
3. A plugin MUST contain zero or more supported components. A directory with only a manifest is valid but may not be useful on hosts that require at least one supported component at runtime.
4. All relative paths declared by the plugin MUST be interpreted relative to the plugin root.
5. All relative paths declared by the plugin MUST start with `./`.
6. A plugin MUST NOT reference files outside its own directory tree by using `../` traversal.
7. Hosts MUST reject any configured path that escapes the plugin root after lexical path normalization or filesystem resolution.
8. When resolving a configured path, hosts MUST treat the filesystem-resolved plugin root as the containment boundary. Symlinks, junctions, reparse points, and equivalent filesystem mechanisms MAY resolve to targets within that boundary, but hosts MUST reject paths that resolve outside it.

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
├── plugin.json
├── skills/
├── .mcp.json
├── LICENSE
└── CHANGELOG.md
```

Example: Core Profile layout (minimal)

```text
code-assistant/
├── plugin.json
├── skills/
│   └── summarize/
│       └── SKILL.md
└── .mcp.json
```

Example: plugin with both v1 component types

```text
devtools/
├── plugin.json
├── skills/
│   └── code-review/
│       ├── SKILL.md
│       ├── scripts/
│       │   └── analyze.sh
│       └── references/
│           └── checklist.md
├── .mcp.json
├── LICENSE
└── CHANGELOG.md
```

<!-- DISCUSSION: standard-layout-extensions — Should the standard layout include additional well-known directories such as `tests/` or `docs/`? -->

> **See also:** [§5 Manifest location and extension fields](#5-manifest-location-and-extension-fields) for manifest location and vendor extension rules, and [§7 Component discovery](#7-component-discovery) for how component directories are scanned.

### 4.3 Directory rules

1. The manifest file MUST be named `plugin.json` and live at the plugin root.
2. Component directories such as `skills/`, when present, MUST exist at the plugin root.
3. Missing component directories are not errors.

Example:

```text
my-plugin/
├── plugin.json
└── skills/
    └── summarize/
        └── SKILL.md
```

## 5. Manifest location and extension fields

### 5.1 Manifest location

Hosts MUST check for a manifest at `plugin.json` in the plugin root.

The Open Plugin core specification defines one manifest file per plugin. Host-specific behavior SHOULD be expressed through namespaced top-level fields inside `plugin.json`, not by adding additional manifest files or vendor-prefixed manifest directories.

| Path          | Description             | Notes                                  |
| ------------- | ----------------------- | -------------------------------------- |
| `plugin.json` | Vendor-neutral manifest | REQUIRED. Lives at the plugin root.    |

Example:

```text
my-plugin/
├── plugin.json
└── skills/summarize/SKILL.md
```

A host selects `plugin.json` and then applies any extension fields it understands.

> **See also:** [§6 Manifest schema](#6-manifest-schema) for the structure of the manifest file, and [§12 Host conformance](#12-host-conformance) for requirements around supporting `plugin.json`.

### 5.2 Vendor extension fields

Implementations MAY define namespaced top-level manifest fields for behavior that is not part of the core specification.

1. Extension fields SHOULD be top-level objects.
2. Extension field names SHOULD identify the implementation, vendor, host family, or standards group that owns the extension.
3. Hosts MUST ignore extension fields they do not understand.
4. Extension fields MUST NOT redefine the semantics of core fields.
5. Extensions that become useful across implementations SHOULD be proposed for promotion into the core specification through the governance process.

Example:

```json
{
  "id": "https://github.com/acme/devtools/tree/main",
  "name": "devtools",
  "version": "2.0.0",
  "openai": {
    "sharing": {
      "workspaceInstall": true
    }
  },
  "vscode": {
    "activationEvents": ["onLanguage:typescript"]
  }
}
```

> **Implementer note:**
> Example user-facing message: `Plugin "devtools": ignoring unsupported extension field "vscode".`
>
> Example machine-readable record:
> ```json
> {"level":"info","event":"open_plugin.manifest.unsupported_extension","plugin":"devtools","field":"vscode","action":"ignored"}
> ```

## 6. Manifest schema

> **See also:** [§5 Manifest location and extension fields](#5-manifest-location-and-extension-fields) for where the manifest is loaded from, and [§7 Component discovery](#7-component-discovery) for how manifest fields control discovery paths.

### 6.1 Manifest object

The manifest MUST be JSON and MUST contain a top-level object.

```json
{
  "id": "https://github.com/example/plugin/tree/main",
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
  "skills": "./custom/skills/",
  "mcpServers": "./mcp-config.json"
}
```

Example: minimal manifest

```json
{
  "id": "https://github.com/example/minimal-plugin/tree/main",
  "name": "minimal-plugin",
  "version": "1.0.0",
  "description": "The simplest possible plugin."
}
```

Example: full manifest

```json
{
  "id": "https://github.com/open-plugin-examples/devtools/tree/main",
  "name": "devtools",
  "version": "2.0.0",
  "description": "Full-featured development toolkit.",
  "author": {
    "name": "Open Plugin Examples"
  },
  "license": "Apache-2.0",
  "keywords": ["devtools", "code-review", "linting", "security"],
  "skills": {
    "paths": ["./skills/"]
  },
  "mcpServers": "./.mcp.json"
}
```

### 6.2 Required fields

| Field  | Type   | Description                                                      |
| ------ | ------ | ---------------------------------------------------------------- |
| `id`   | string | Durable plugin identifier. SHOULD be a URL for the canonical plugin location. |
| `name` | string | Human-readable plugin identifier used for namespacing and settings keys. |

The `id` value MUST be non-empty. If `id` or `name` is missing, has the wrong type, is empty, or otherwise violates its requirements, the manifest is invalid. Hosts MUST reject the plugin and MUST NOT discover or execute any of its components. Hosts SHOULD report which required field is invalid.

The `id` field is intended for provenance, attribution, policy, and update tracking. The simplest portable form is the canonical URL where the plugin can be retrieved, such as a GitHub repository path or a cloud-hosted plugin URL.

Examples:

```json
{
  "id": "https://github.com/acme/agent-plugins/tree/main/plugins/devtools",
  "name": "devtools"
}
```

```json
{
  "id": "https://plugins.example.com/acme/devtools",
  "name": "devtools"
}
```

Hosts MAY store content hashes, resolved commit SHAs, signatures, or other integrity metadata alongside the plugin, but those mechanisms are not required by this specification.

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

## 7. Component discovery

> **See also:** [§4 Plugin package model](#4-plugin-package-model) for directory layout conventions, [§6 Manifest schema](#6-manifest-schema) for how manifest fields declare discovery paths, and [§9 Namespacing](#9-namespacing) for how discovered components are named.

### 7.1 Default locations

Hosts MUST discover core components in the following default locations when no manifest field overrides discovery for that component type. When the manifest declares paths for a component type, those paths control discovery and the default location is not scanned unless explicitly included. See [§6.4](#64-component-path-fields) for details.

Core component locations:

| Component     | Default location   | Default pattern                      |
| ------------- | ------------------ | ------------------------------------ |
| Skills        | `skills/`          | Subdirectories containing `SKILL.md` |
| MCP servers   | `.mcp.json`        | JSON configuration                   |

Example: given a plugin `reports-plugin` with this layout:

```text
reports-plugin/
├── plugin.json
├── skills/summarize/SKILL.md
└── .mcp.json
```

The host discovers skill `summarize` and MCP servers from `.mcp.json` — all from default locations.

### 7.2 Missing locations

1. If a default location is absent, the host MUST NOT treat that as an error.
2. When the manifest declares paths for a component type, those paths control discovery. The default location is not scanned unless explicitly included.
3. When no manifest field is declared for a component type, the default location is scanned normally.

Example: a plugin declares `"skills": "./custom-skills/"` but has no `skills/` directory. The host scans only `custom-skills/` and does not error on the missing default `skills/` path.

### 7.3 Discovery examples

Example: default discovery only

```text
reports-plugin/
├── plugin.json
├── skills/summarize/SKILL.md
└── .mcp.json
```

The host discovers skill `summarize` and MCP servers from `.mcp.json` — all from default locations.

Example: manifest paths override defaults

`./plugin.json`

```json
{
  "id": "https://github.com/example/reports-plugin/tree/main",
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

`./plugin.json`

```json
{
  "id": "https://github.com/example/reports-plugin/tree/main",
  "name": "reports-plugin",
  "skills": ["./skills/", "./custom-skills/"]
}
```

The host discovers both `/reports-plugin:summarize` and `/reports-plugin:deploy` because the default directory is explicitly included.

## 8. Component definitions

> **See also:** [§7 Component discovery](#7-component-discovery) for how component files are located, and [§9 Namespacing](#9-namespacing) for how component names are surfaced to users and models.

Open Plugin v1 defines exactly two portable component types: **skills** and **MCP servers**. Other capabilities are outside the v1 format and do not affect conformance.

Hosts MUST ignore component types they do not support.

### 8.1 Skills

Agent Skills MUST conform to the [Agent Skills specification](https://agentskills.io/specification). That specification is the source of truth for the `SKILL.md` format, frontmatter fields, and directory layout (`scripts/`, `references/`, `assets/`).

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

This specification defines how MCP servers are *discovered* within a plugin and how `${PLUGIN_ROOT}` and `${PLUGIN_DATA}` expansion applies to configuration values.

#### 8.2.1 Discovery and configuration

The default MCP configuration path is `.mcp.json`. MCP servers MAY also be declared inline in the manifest `mcpServers` field.

The configuration MUST contain a top-level `mcpServers` object. Each key is the server name and MUST be unique within the plugin after source resolution.

The `command` field MUST contain a single executable token, not a shell command string. It MUST be either a bare executable name or a plugin-relative path beginning with `./`. Hosts MUST resolve bare names using the platform's executable search rules and MUST resolve plugin-relative paths against the plugin root. Hosts MUST NOT perform placeholder expansion in `command`.

Hosts MAY use a platform-specific command interpreter when required to launch the resolved executable, such as a `.bat` or `.cmd` script on Windows, but MUST preserve `command` as one token and pass `args` separately. When `cwd` is omitted, hosts MUST use the plugin root as the subprocess working directory.

The `args`, `env`, and `cwd` fields in MCP server configuration MUST support `${PLUGIN_ROOT}` and `${PLUGIN_DATA}` expansion.

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
      "command": "./bin/fs-server",
      "args": ["--root", "${PLUGIN_DATA}/filesystem"]
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

MCP tool identifiers surfaced by a host are outside the Open Plugin package format and are not part of plugin or host conformance.

Implementer note: A host that aggregates tools from multiple MCP servers may encounter duplicate tool names. The host should use a deterministic disambiguation strategy and retain an unambiguous mapping from each surfaced identifier to the originating plugin, server, and tool. Any transformed identifier should satisfy the tool-name constraints of the MCP version implemented by the host.

Example: component namespacing

| Source file | Component name | Surfaced identifier |
| --- | --- | --- |
| `skills/code-review/SKILL.md` | `code-review` | `devtools:code-review` |
| `skills/deploy/SKILL.md` | `deploy` | `devtools:deploy` |

## 10. Environment variables and placeholder expansion

> **See also:** [§8.2 MCP servers](#82-mcp-servers) for the fields where plugin variable expansion applies, and [§4.1 General requirements](#41-general-requirements) for path safety rules.

### 10.1 Required variables

Hosts that launch plugin subprocesses (i.e., MCP servers) MUST provide an environment variable containing the absolute plugin root path.

| Variable      | Description             | Notes                                                      |
| ------------- | ----------------------- | ---------------------------------------------------------- |
| `PLUGIN_ROOT` | Absolute plugin root path | REQUIRED for hosts that launch plugin subprocesses. |

### 10.1.1 Persistent data directory

Hosts that launch plugin subprocesses MUST provide a dedicated writable data directory for each plugin and expose its absolute path through `PLUGIN_DATA`.

| Variable      | Description                    | Notes                                             |
| ------------- | ------------------------------ | ------------------------------------------------- |
| `PLUGIN_DATA` | Persistent plugin data directory | REQUIRED for hosts that launch plugin subprocesses. |

`PLUGIN_DATA` is the absolute path to a host-managed persistent data directory dedicated to the plugin. The host MUST create the directory before launching a plugin subprocess, MUST make it writable to that subprocess, and MUST preserve its contents across plugin updates. The host MAY delete the directory when the plugin is uninstalled.

Use `PLUGIN_DATA` for: installed dependencies (node_modules, virtual environments), generated code, caches, and other plugin state that should persist across updates. Use `PLUGIN_ROOT` for referencing bundled scripts, binaries, and config files that ship with the plugin.

Example: a host loading the plugin `devtools` from `/home/alex/.agents/plugins/devtools` sets:

```text
PLUGIN_ROOT=/home/alex/.agents/plugins/devtools
PLUGIN_DATA=/home/alex/.agents/plugins/data/devtools
```

### 10.2 Placeholder expansion

Hosts that launch plugin subprocesses MUST expand `${PLUGIN_ROOT}` and `${PLUGIN_DATA}` in supported configuration fields.

Plugins claiming conformance MUST NOT require interpolation of variables other than `${PLUGIN_ROOT}` and `${PLUGIN_DATA}`. Hosts MAY support additional variables as host-specific behavior.

Expansion applies to runtime arguments, environment-bearing strings, and explicit working directories, not to `command` or manifest discovery path fields. Manifest discovery paths (§6.4) are always relative `./` paths resolved from the plugin root.

| Location                     | Fields                                                                                       | Description                                            |
| ---------------------------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| MCP config                   | `args`, `env`, `cwd`                                                                         | All string values support plugin variable expansion.   |

Hosts MAY provide additional environment variables beyond the plugin root variables.

Example: plugin variable expansion in MCP

```json
{
  "mcpServers": {
    "database": {
      "command": "npx",
      "args": ["--config", "${PLUGIN_ROOT}/config/db.json"],
      "cwd": "${PLUGIN_ROOT}",
      "env": {
        "DATA_DIR": "${PLUGIN_DATA}/database"
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
2. Parses `plugin.json`.
3. For each core component type it supports (skills, MCP servers), discovers components in default locations.
4. Respects manifest-declared discovery paths for supported component types.
5. If the host launches plugin subprocesses (e.g., MCP servers), provides `PLUGIN_ROOT` and `PLUGIN_DATA` and expands both variables in runtime configuration values (`args`, `env`, `cwd`).
6. For MCP servers, resolves `command` as a single executable token and uses the plugin root as the default subprocess working directory.
7. Supports at least one core component type (skills or MCP servers).

Host-specific behavior should be represented by namespaced extension fields in `plugin.json`. This keeps the plugin package inspectable through one canonical manifest while allowing implementations to experiment independently.

Example: a skills-only host is conformant. It only needs to:

```text
1. Accept a plugin directory path.
2. Read plugin.json for the plugin id and name.
3. Scan skills/ for SKILL.md files (default location discovery).
4. Respect manifest-declared skill paths.
```

A host that only supports skills and ignores MCP servers is fully conformant to Open Plugin v1 as long as it meets all seven requirements above.

### 12.2 Incremental adoption

A host is not required to support every component type. Incremental adoption is conformant.

### 12.3 Unsupported components and failures

1. Hosts MUST ignore unsupported component types.
2. Hosts MUST continue loading a plugin when a component fails independently, such as an MCP server startup failure.
3. Hosts SHOULD warn when configuration is invalid, conflicting, or partially unsupported.

---

## Appendix A: Conformance Checklist

*This checklist is for convenience only — when it conflicts with the spec text above, the spec governs.*

### Plugin loader

- [ ] Parse `plugin.json` ([§5.1](#51-manifest-location))
- [ ] Validate required `id` and `name` fields ([§6.2](#62-required-fields))
- [ ] Validate plugin name against naming constraints ([§6.7](#67-plugin-name-constraints))
- [ ] Reject paths that escape the plugin root via `../` ([§4.1](#41-general-requirements))
- [ ] Ignore unsupported vendor extension fields ([§5.2](#52-vendor-extension-fields))

### Component discovery

- [ ] Scan default locations for each supported component type ([§7.1](#71-default-locations))
- [ ] Respect manifest-declared discovery paths ([§6.4](#64-component-path-fields), [§7.2](#72-missing-locations))
- [ ] Ignore missing default directories without error ([§7.2](#72-missing-locations))

### Namespacing

- [ ] SHOULD namespace components as `{plugin-name}:{component-name}` ([§9.1](#91-general-component-namespacing))

### Environment and expansion

- [ ] If the host launches plugin subprocesses, provide `PLUGIN_ROOT` environment variable ([§10.1](#101-required-variables))
- [ ] If the host launches plugin subprocesses, provide a dedicated writable `PLUGIN_DATA` directory ([§10.1.1](#1011-persistent-data-directory))
- [ ] Resolve MCP server `command` as a single bare or plugin-relative executable token ([§8.2.1](#821-discovery-and-configuration))
- [ ] Use the plugin root as the default MCP server working directory ([§8.2.1](#821-discovery-and-configuration))
- [ ] Expand `${PLUGIN_ROOT}` and `${PLUGIN_DATA}` in MCP server `args`, `env`, and `cwd` fields ([§10.2](#102-placeholder-expansion))

### Resilience

- [ ] Ignore unsupported component types ([§12.3](#123-unsupported-components-and-failures))
- [ ] Continue loading when an independent component fails ([§12.3](#123-unsupported-components-and-failures))
- [ ] Support at least one core component type ([§12.1](#121-minimum-host-requirements))

### Diagnostics matrix

*This table consolidates the failure-handling behaviors defined throughout the spec into a single reference. All behaviors listed below restate existing requirements — no new requirements are introduced. Host implementers can use this matrix to verify that their diagnostic output covers every specified failure site.*

<!-- DISCUSSION: diagnostics-contract -->

| Decision point | Existing spec behavior | Human-readable example | JSON example | Fatal? | Verification |
| --- | --- | --- | --- | --- | --- |
| Unsupported extension field | Hosts MUST ignore extension fields they do not understand ([§5.2](#52-vendor-extension-fields)) | `INFO open-plugin: plugin "devtools" declares unsupported extension field "vscode"; ignored` | `{"level":"info","event":"open_plugin.manifest.unsupported_extension","plugin":"devtools","field":"vscode","action":"ignored"}` | No | Check that core fields and supported extensions still load |
| Missing or invalid required field | Hosts MUST reject a plugin whose `id` or `name` is missing or invalid and MUST NOT discover or execute its components ([§6.2](#62-required-fields)) | `ERROR open-plugin: manifest is invalid: required field "id" is missing; plugin rejected` | `{"level":"error","event":"open_plugin.manifest.invalid_required_field","field":"id","reason":"missing","action":"rejected"}` | Yes | Check that no plugin components are discovered or executed |
| Invalid ambiguous object field | Hosts SHOULD treat unrecognized object shapes as invalid and SHOULD warn ([§6.6](#66-object-field-disambiguation)) | `WARN open-plugin: plugin "devtools" manifest field "mcpServers" is invalid: expected either a path config with "paths" key or an inline config with "mcpServers" key; field ignored, plugin load continues` | `{"level":"warn","event":"open_plugin.manifest.invalid_object","plugin":"devtools","field":"mcpServers","action":"ignored","continue":true}` | No | Check that the invalid field is skipped and remaining components load |
| MCP server startup failure | If a server fails to start, the host SHOULD log the error and continue loading other components ([§8.2.2](#822-plugin-specific-discovery-rules)) | `ERROR open-plugin: plugin "devtools" MCP server "database" failed to start: connection refused on port 5432. Other plugin components remain available.` | `{"level":"error","event":"open_plugin.mcp.start_failed","plugin":"devtools","server":"database","error":"connection refused on port 5432","action":"continue_without_mcp"}` | No | Check that other components still load |
| MCP server name conflict | Hosts SHOULD warn and SHOULD resolve conflicts deterministically. Conflicting names MUST NOT crash plugin loading ([§8.2.2](#822-plugin-specific-discovery-rules)) | `WARN open-plugin: plugin "devtools" MCP server name "filesystem" defined in multiple config files; using first definition` | `{"level":"warn","event":"open_plugin.mcp.name_conflict","plugin":"devtools","server":"filesystem","action":"used_first"}` | No | Check that one definition wins deterministically and loading continues |
| Partial host support | Hosts SHOULD warn when configuration is invalid, conflicting, or partially unsupported ([§12.3](#123-unsupported-components-and-failures)) | `WARN open-plugin: plugin "devtools" is partially supported: this host supports skills but not MCP servers` | `{"level":"warn","event":"open_plugin.host.partial_support","plugin":"devtools","supported":["skills"],"unsupported":["mcpServers"],"action":"loaded_partial"}` | No | Check that supported components are functional |

> **Implementer note:** The `event` field values above (e.g., `open_plugin.manifest.invalid_object`) are *suggested* stable identifiers — not required by this spec. Hosts that adopt them gain a machine-readable diagnostic surface that agents, CI pipelines, and plugin validators can consume deterministically. The recommended fields for every diagnostic record are: `level`, `event`, `plugin` (plugin name), the relevant component identifier (e.g., `server` or `field`), and `action` (what the host did in response).

---

## Design Decisions

*This section explains why key design choices were made. It is for context only — the binding rules are in sections 1–12 above.*

### Why directory-based discovery?

Plugins use filesystem directories as the package unit rather than archive formats (`.zip`, `.tar.gz`) or registry-fetched bundles. This keeps plugins inspectable with standard tools (`ls`, `cat`, `git`), editable in-place during development, and compatible with version control without special tooling.

### Why only Agent Skills and MCP in v1?

Agent Skills and MCP have independently maintained formats with meaningful cross-host adoption. Other proposed component types — such as commands, hooks, agents, rules, and LSP servers — remain too host-specific for a stable portable contract and are outside portable v1 until their formats converge.

### Why colon-separated namespacing for components?

The `plugin-name:component-name` format was chosen because colons are visually distinct and rarely appear in component names. Alternatives considered included `/` (conflicts with filesystem paths) and `__` (less readable for user-facing identifiers).

### Why root-level `plugin.json` is the conformance floor

Every conformant host MUST check `plugin.json` at the plugin root ([§5.1](#51-manifest-location)). This gives plugin authors a single guaranteed file that works across all hosts without vendor-specific path knowledge. Vendor-specific behavior belongs in namespaced extension fields, so custom behavior is visible in the same manifest as the portable core fields.

### Why one manifest instead of per-vendor files?

One manifest avoids scattering plugin configuration across multiple directories and makes it easier for authors, hosts, and reviewers to see which fields are shared and which fields are implementation-specific. Top-level extension objects provide room for host-specific experimentation without creating competing manifest precedence rules.

### Why plugin variables over relative paths in configs?

MCP server arguments often need absolute paths at runtime. `${PLUGIN_ROOT}` provides an unambiguous, host-resolved anchor for bundled files, while `${PLUGIN_DATA}` identifies host-managed writable state that persists when package contents are replaced during an update. The `command` field does not use interpolation: a `./` path is resolved directly against the plugin root, and a bare name uses the platform's executable search rules. Treating `command` as one token avoids requiring hosts to parse and escape user-authored shell command strings.

### Why component failures are non-fatal

When an MCP server fails to start, the host continues loading the plugin's remaining components ([§12.3](#123-unsupported-components-and-failures)). A plugin that provides skills and an MCP server should not become entirely unusable because the server's runtime dependency is missing or its port is occupied. The spec pairs non-fatal component failures with diagnostic requirements so that failures are visible rather than silent.

<!-- DISCUSSION: diagnostics-contract -->

---

## Future Considerations

*Everything in this section is deferred from v1.0.0. Nothing here is required for conformance. These items may be addressed in future versions.*

### Permission and approval UX

v1.0.0 does not define a trust model, permission system, or sandboxing requirements for plugins. A future version should address:

- Permission declarations in the manifest (e.g., filesystem access, network access, tool access)
- Host-enforced capability restrictions per plugin
- User consent flows for plugin installation and capability grants
- Approval UX for MCP servers that execute arbitrary commands or access external services
- Graduated trust levels (e.g., "sandboxed", "user-approved", "organization-approved")

### Provenance verification

v1.0.0 does not specify how hosts or users can verify the origin or integrity of a plugin. A future version may define:

- Cryptographic signature verification for published plugins
- Attestation chains linking a published plugin to its source repository and build
- Host-side policies for requiring signatures from trusted publishers

### Secret and sensitive value handling

MCP servers often need credentials or API keys at runtime. v1.0.0 does not specify how sensitive values should be provided, stored, or scoped. A future version may define:

- A `secrets` manifest field or separate secrets configuration
- Host-mediated secret injection that avoids plaintext in config files
- Scoping rules that prevent one plugin from accessing another plugin's secrets
- Rotation and revocation semantics for plugin-held credentials

### Enterprise controls

Organizations deploying plugins at scale need policy enforcement that v1.0.0 does not address. A future version may define:

- Allowlist and blocklist policies for plugin installation by name, publisher, or signature
- Organization-scoped plugin registries with approval workflows
- Centralized configuration overrides that take precedence over user-level plugin settings
- Compliance reporting for plugin installation and usage events

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

- Precompiled binary distribution for MCP servers
- Platform-specific binary selection
- Integrity verification for binary artifacts

### Plugin testing and validation

No test harness or validation tool is specified. A future version may define:

- A `test` manifest field or convention
- A standard plugin linter or validator command
- Conformance test suites for host implementations
