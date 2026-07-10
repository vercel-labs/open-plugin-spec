# Open Plugin Specification

**Spec Version: 1.0.0**

This document defines the canonical Open Plugin Specification v1.0.0. It is a self-contained specification for packaging agent extensions into distributable plugins. Everything in sections 1–11 is required for conformance.

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

Greet the user and offer help.
```

A host that supports skills can load this plugin by reading `plugin.json` and discovering `skills/greet/SKILL.md`. How the host exposes the skill to users or models is outside this specification.

> **Note:**
> Open Plugin v1 standardizes two component types: Agent Skills and MCP servers. Other capabilities are outside the portable v1 format.

## Table of contents

1. [Status and version](#1-status-and-version)
2. [Conformance language](#2-conformance-language)
3. [Terminology](#3-terminology)
4. [Plugin package model](#4-plugin-package-model)
5. [Manifest](#5-manifest)
6. [Component discovery](#6-component-discovery)
7. [Component definitions](#7-component-definitions)
8. [Client extensions](#8-client-extensions)
9. [Environment variables and placeholder expansion](#9-environment-variables-and-placeholder-expansion)
10. [Versioning](#10-versioning)
11. [Host conformance](#11-host-conformance)

**Appendices (not required for conformance)**

- [Appendix A: Conformance Checklist](#appendix-a-conformance-checklist)
- [Design Decisions](#design-decisions)
- [Future Considerations](#future-considerations)

## 1. Status and version

This specification defines version `1.0.0` of the Open Plugin format.

Plugin hosts and plugin packages claiming conformance to Open Plugin v1 MUST implement or follow the requirements in this document.

### 1.1 Governance model

Open Plugin project governance is defined separately from the portable package format in the [Technical Charter](./GOVERNANCE.md).

## 2. Conformance language

The key words MUST, MUST NOT, REQUIRED, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL in sections 1–11 of this document are to be interpreted as described in RFC 2119 and RFC 8174 when, and only when, they appear in all capitals. Appendices and other non-normative sections use these terms informally.

## 3. Terminology

| Term               | Meaning                    | Description                                                                                                                                  |
| ------------------ | -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| Plugin             | Package unit               | A self-contained directory that bundles one or more components and optional metadata.                                                        |
| Plugin root        | Filesystem root            | The top-level directory of a plugin package.                                                                                                 |
| Manifest           | Metadata document          | A `plugin.json` file at the plugin root.                                                                                                    |
| Component          | Plugin-provided capability | A skill or MCP server configuration.                                                                                                         |
| Host               | Plugin runtime             | A tool that discovers, installs, loads, and executes plugin components.                                                                      |
| Client namespace   | Extension directory        | A dot-prefixed top-level directory containing behavior owned by one client.                                                                  |

## 4. Plugin package model

### 4.1 General requirements

1. A plugin is a directory rooted at a single filesystem location.
2. A plugin MUST include a manifest at `plugin.json` in the plugin root.
3. A plugin MUST contain zero or more supported components. A directory with only a manifest is valid but may not be useful on hosts that require at least one supported component at runtime.
4. When a host discovers, reads, or executes a file or directory supplied by the plugin package, the filesystem-resolved path MUST remain within the filesystem-resolved plugin root. Symlinks, junctions, reparse points, and equivalent filesystem mechanisms MAY resolve to targets within the plugin root, but hosts MUST reject package paths that resolve outside it.
5. A configuration field defined by this specification as a plugin-relative path MUST begin with `./`, be resolved against the plugin root, and remain within the filesystem-resolved plugin root after resolution.
6. Configuration values not defined as paths, including command arguments and environment variable values, are opaque strings. Hosts MUST NOT interpret them as package paths for the purpose of enforcing this section.

Example: valid and invalid relative paths

```json
{
  "mcpServers": {
    "server": {
      "type": "stdio",
      "command": "./bin/server",
      "cwd": "./data"
    }
  }
}
```

```json
{
  "mcpServers": {
    "server": {
      "type": "stdio",
      "command": "../bin/server",
      "cwd": "data"
    }
  }
}
```

The first example is valid — both paths start with `./` and stay within the plugin root. The second is invalid — `../bin/server` escapes the plugin root and `data` is not a plugin-relative path.

These containment rules govern access to files supplied by the plugin package. They do not sandbox a plugin subprocess or restrict paths supplied at runtime, including the host-managed `PLUGIN_DATA` directory defined in §9.1.

### 4.2 Standard layout

A plugin that uses both portable component types and a client extension can have the following layout:

```text
my-plugin/
├── plugin.json
├── skills/
│   └── summarize/
│       ├── SKILL.md
│       ├── scripts/
│       │   └── analyze.sh
│       └── references/
│           └── checklist.md
├── mcp.json
├── .client-name/
├── LICENSE
└── CHANGELOG.md
```

> **See also:** [§5 Manifest](#5-manifest) for manifest rules, [§6 Component discovery](#6-component-discovery) for fixed component locations and missing-location behavior, and [§8 Client extensions](#8-client-extensions) for client namespace rules.

## 5. Manifest

### 5.1 Location and loading

Hosts MUST check for a manifest at `plugin.json` in the plugin root.

The Open Plugin core specification defines exactly one portable manifest per plugin. No other file can replace, supplement, or override the core fields in root `plugin.json`.

A host loads and validates root `plugin.json` before discovering components or applying client-specific behavior.

> **See also:** [§11 Host conformance](#11-host-conformance) for requirements around supporting `plugin.json`.

### 5.2 Manifest object

The manifest MUST be JSON and MUST contain a top-level object. Its schema is closed: the only permitted top-level fields are `id`, `name`, `version`, `description`, `author`, `homepage`, `repository`, `license`, and `keywords`.

If `plugin.json` contains any other top-level field, the manifest is invalid. Hosts MUST reject the plugin and MUST NOT discover or execute any of its components. Hosts SHOULD report each unsupported field. Client-specific fields belong under `.<client>/` as defined in §8.

Every permitted field MUST match the type and constraints defined below. Any schema violation makes the manifest invalid and requires the same rejection behavior.

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
  "keywords": ["keyword1", "keyword2"]
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
  "keywords": ["devtools", "code-review", "linting", "security"]
}
```

### 5.3 Required fields

| Field  | Type   | Description                                                      |
| ------ | ------ | ---------------------------------------------------------------- |
| `id`   | string | Durable plugin identifier. SHOULD be a URL for the canonical plugin location. |
| `name` | string | Human-readable plugin identifier. |

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

### 5.4 Metadata fields

| Field         | Type     | Description                                                           |
| ------------- | -------- | --------------------------------------------------------------------- |
| `version`     | string   | Version string (Semantic Versioning RECOMMENDED). Used for update checks and cache freshness. |
| `description` | string   | Short description of plugin purpose.                                  |
| `author`      | object   | Author object with optional `name`, `email`, and `url` string fields. |
| `homepage`    | string   | Documentation or homepage URL.                                        |
| `repository`  | string   | Source repository URL.                                                |
| `license`     | string   | License identifier (SPDX identifier RECOMMENDED).                     |
| `keywords`    | string[] | Search and discovery tags.                                            |

The `author` object MAY contain only the `name`, `email`, and `url` fields, each with a string value. Any other field or value type makes the manifest invalid.

Except where this specification states an explicit constraint, metadata fields are validated only by their JSON types. Hosts MUST NOT reject a manifest solely because `version` is not valid Semantic Versioning; `homepage`, `repository`, or `author.url` is not a recognized URL; `author.email` is not a recognized email address; or `license` is not an SPDX identifier.

### 5.5 Plugin name constraints

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

## 6. Component discovery

> **See also:** [§4 Plugin package model](#4-plugin-package-model) for directory layout conventions.

### 6.1 Fixed locations

Hosts MUST discover each supported component type from its fixed location. `plugin.json` cannot override these locations or contain inline component configuration.

Core component locations:

| Component     | Fixed location     | Pattern                              |
| ------------- | ------------------ | ------------------------------------ |
| Skills        | `skills/`          | Subdirectories containing `SKILL.md` |
| MCP servers   | `mcp.json`         | JSON configuration                   |

Example: given a plugin `reports-plugin` with this layout:

```text
reports-plugin/
├── plugin.json
├── skills/summarize/SKILL.md
└── mcp.json
```

The host discovers skill `summarize` from `skills/` and MCP servers from `mcp.json`.

### 6.2 Missing locations

If a fixed component location is absent, the host MUST NOT treat that as an error.

If a fixed component location is present but does not resolve to the expected filesystem kind — for example, `skills` does not resolve to a directory or `mcp.json` does not resolve to a regular file — the host MUST treat that component type as invalid and continue loading other supported component types.

## 7. Component definitions

> **See also:** [§6 Component discovery](#6-component-discovery) for how component files are located.

Open Plugin v1 defines exactly two portable component types: **skills** and **MCP servers**. Other capabilities are outside the v1 format and do not affect conformance.

Hosts MUST ignore component types they do not support.

### 7.1 Skills

Agent Skills MUST conform to the [Agent Skills specification](https://agentskills.io/specification). That specification is the source of truth for the `SKILL.md` format, frontmatter fields, and directory layout (`scripts/`, `references/`, `assets/`).

This specification defines how Agent Skills are *discovered* within a plugin, not the skill format itself or how hosts expose skills to users or models.

The fixed discovery location is `skills/`. Each immediate child directory containing a path named exactly `SKILL.md` that resolves to a regular file is treated as one skill. Hosts MUST NOT recursively search deeper descendants for additional skills.

If a discovered skill does not conform to the Agent Skills specification, the host MUST skip that skill and continue loading other skills and component types. The host SHOULD report the invalid skill.

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

### 7.2 MCP servers

The [Model Context Protocol specification](https://modelcontextprotocol.io/specification) defines MCP wire behavior and lifecycle semantics. Open Plugin defines the `mcp.json` configuration format used to locate and connect to MCP servers in a plugin. Hosts map this portable format to their native configuration; its field names and values need not match a host-native format.

#### 7.2.1 Discovery and configuration

The MCP configuration path is `mcp.json` at the plugin root. MCP configuration MUST NOT be declared inline in `plugin.json` or loaded from any alternative core path.

`mcp.json` MUST be a JSON object containing exactly one field, `mcpServers`. `mcpServers` MUST be an object whose member names identify servers and whose member values are server configuration objects. An empty `mcpServers` object is valid.

Each server configuration MUST contain a `type` field and match exactly one of the closed variants below. An unknown field, an unknown `type` value, or a field belonging to another variant makes that server entry invalid.

##### stdio

| Field     | Type              | Required | Description                                      |
| --------- | ----------------- | -------- | ------------------------------------------------ |
| `type`    | `"stdio"`         | Yes      | Selects the MCP stdio transport.                 |
| `command` | string            | Yes      | Executable token to launch.                      |
| `args`    | string[]          | No       | Arguments passed to the executable.              |
| `env`     | object of strings | No       | Environment variables supplied to the process.   |
| `cwd`     | string            | No       | Working directory for the process.               |

The `command` field MUST contain a single executable token, not a shell command string. It MUST be either a bare executable name or a plugin-relative path beginning with `./`. Hosts MUST resolve bare names using the platform's executable search rules and MUST resolve plugin-relative paths against the plugin root. Hosts MUST NOT perform placeholder expansion in `command`.

Hosts MAY use a platform-specific command interpreter when required to launch the resolved executable, such as a `.bat` or `.cmd` script on Windows, but MUST preserve `command` as one token and pass `args` separately. When `cwd` is omitted, hosts MUST use the plugin root as the subprocess working directory.

The `args`, `env`, and `cwd` fields in a stdio server configuration MUST support `${PLUGIN_ROOT}` and `${PLUGIN_DATA}` expansion.

##### Streamable HTTP and legacy HTTP+SSE

| Field     | Type                          | Required | Description                                                       |
| --------- | ----------------------------- | -------- | ----------------------------------------------------------------- |
| `type`    | `"streamable-http"` or `"sse"` | Yes      | Selects the remote MCP transport.                                 |
| `url`     | string                        | Yes      | MCP endpoint URL.                                                 |
| `headers` | object of strings             | No       | Fixed HTTP headers sent when connecting to the configured origin. |

`streamable-http` selects the current MCP Streamable HTTP transport. `sse` selects the deprecated HTTP+SSE transport defined by the [MCP 2024-11-05 specification](https://modelcontextprotocol.io/specification/2024-11-05/basic/transports); it does not refer to SSE responses or streams used within Streamable HTTP.

The `url` value MUST be an absolute HTTP or HTTPS URL and MUST NOT contain user information or a fragment. Non-loopback endpoints MUST use HTTPS. HTTP MAY be used when the URL host is exactly `localhost` or an IP literal in a loopback range.

Header names and values MUST be valid HTTP header fields. Header names are case-insensitive; an entry containing the same header name more than once under different casing is invalid. Hosts MUST NOT perform placeholder or environment-variable expansion in `url`, header names, or header values.

Header values are visible package data, not a portable secret mechanism. Plugins MUST NOT embed credentials or other secrets in `headers`. Headers generated by the host to implement HTTP, MCP, or authorization take precedence over configured headers with the same case-insensitive name. A host MUST NOT forward configured headers to a different origin through a redirect or legacy SSE endpoint event without explicit user authorization.

Open Plugin v1 defines no OAuth configuration or portable credential-reference fields. Authorization discovery, user interaction, and credential storage are host-managed. An authorization failure is a connection failure for that server, not invalid plugin configuration.

##### Transport support

A host that supports Open Plugin MCP servers MUST support both `stdio` and `streamable-http`. Support for `sse` is OPTIONAL. A host MUST use the transport declared by `type` and MUST NOT reinterpret `streamable-http` as legacy HTTP+SSE or automatically fall back to it.

Example: `mcp.json`

```json
{
  "mcpServers": {
    "local-validator": {
      "type": "stdio",
      "command": "./bin/validator",
      "args": ["--data", "${PLUGIN_DATA}/validator"],
      "env": {
        "CONFIG": "${PLUGIN_ROOT}/config.json"
      },
      "cwd": "${PLUGIN_ROOT}"
    },
    "deployment-api": {
      "type": "streamable-http",
      "url": "https://deploy.example.com/mcp",
      "headers": {
        "X-Tenant": "public-tenant"
      }
    },
    "legacy-events": {
      "type": "sse",
      "url": "https://legacy.example.com/sse"
    }
  }
}
```

#### 7.2.2 Loading rules

1. Hosts that support MCP servers MUST load configuration only from `mcp.json` at the plugin root.
2. If `mcp.json` is not valid JSON or does not satisfy the top-level requirements in §7.2.1, the host MUST disable MCP for that plugin and continue loading other component types. The host SHOULD report the invalid configuration.
3. If an individual server entry does not satisfy the requirements in §7.2.1, the host MUST skip that server and continue loading other servers and component types. The host SHOULD report the invalid entry.
4. If the host does not support a valid `sse` entry, it MUST skip that server and continue loading other servers and component types. The host SHOULD report the unsupported transport.
5. If a server fails to start, connect, authenticate, or complete the MCP handshake, the host MUST continue loading other servers and component types. The host SHOULD report the connection failure.

## 8. Client extensions

A client MAY use a top-level `.<client>/` directory for client-specific content. Open Plugin assigns no structure or semantics to the contents of that directory.

1. Client-specific content MUST live inside the namespace owned by that client and MUST NOT add fields to root `plugin.json`.
2. Hosts MUST ignore client namespaces they do not implement.
3. Client namespace contents do not affect Open Plugin conformance.
4. A client MAY define its own precedence rules, including rules that supplement or override portable components in the plugin root. Such behavior is client-specific and outside this specification.
5. Top-level paths not defined by Open Plugin v1 MUST NOT be interpreted as portable component types.

## 9. Environment variables and placeholder expansion

> **See also:** [§7.2 MCP servers](#72-mcp-servers) for the fields where plugin variable expansion applies, and [§4.1 General requirements](#41-general-requirements) for path safety rules.

### 9.1 Required variables

Hosts that launch plugin subprocesses (i.e., stdio MCP servers) MUST provide `PLUGIN_ROOT` and `PLUGIN_DATA` in each subprocess environment. `PLUGIN_ROOT` is the absolute path to the filesystem-resolved plugin root. `PLUGIN_DATA` is the absolute path to a host-managed persistent data directory dedicated to that installed plugin instance.

The host chooses the `PLUGIN_DATA` location. It MUST create the directory before launching a plugin subprocess, MUST make it writable to that subprocess, and MUST preserve its contents across plugin updates. The host MAY delete the directory when the plugin is uninstalled.

Use `PLUGIN_DATA` for: installed dependencies (node_modules, virtual environments), generated code, caches, and other plugin state that should persist across updates. Use `PLUGIN_ROOT` for referencing bundled scripts, binaries, and config files that ship with the plugin.

Example: a host loading the plugin `devtools` from `/home/alex/.agents/plugins/devtools` sets:

```text
PLUGIN_ROOT=/home/alex/.agents/plugins/devtools
PLUGIN_DATA=/home/alex/.agents/plugins/data/devtools
```

### 9.2 Placeholder expansion

Hosts that launch plugin subprocesses MUST expand `${PLUGIN_ROOT}` and `${PLUGIN_DATA}` in supported configuration fields. Expansion is a single, non-recursive textual replacement of every exact occurrence of either placeholder. Text introduced by a replacement MUST NOT be scanned for further placeholders.

Expansion applies to every string element of `args`, every string value in `env`, and the `cwd` string. It does not apply to `env` keys, `command`, or fixed component locations.

Unrecognized placeholder-like text MUST remain literal. Plugins claiming conformance MUST NOT depend on interpolation of placeholders other than `${PLUGIN_ROOT}` and `${PLUGIN_DATA}`.

An MCP server's `env` object MUST NOT contain entries named `PLUGIN_ROOT` or `PLUGIN_DATA`. Such an entry makes that server configuration invalid under §7.2.2. Hosts MUST supply the reserved environment variables themselves.

Hosts MAY provide additional environment variables beyond `PLUGIN_ROOT` and `PLUGIN_DATA`.

Example: plugin variable expansion in MCP

```json
{
  "mcpServers": {
    "database": {
      "type": "stdio",
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

## 10. Versioning

Plugins SHOULD use Semantic Versioning for `version`.

| Segment | Meaning                     | Description                                            |
| ------- | --------------------------- | ------------------------------------------------------ |
| Major   | Breaking change             | Incompatible behavior or schema change.                |
| Minor   | Backward-compatible feature | New behavior without breaking existing hosts or users. |
| Patch   | Backward-compatible fix     | Corrective change without intended behavioral break.   |

Hosts MAY use `version` to determine whether updates are available and whether caches are stale.

## 11. Host conformance

### 11.1 Minimum host requirements

A conformant host MUST satisfy all applicable requirements in sections 1–10. At minimum, it:

1. Can load a plugin from a directory path.
2. Parses and validates the closed `plugin.json` schema.
3. Ignores client extension directories it does not implement.
4. For each core component type it supports, discovers components in its fixed location.
5. If it supports MCP servers, supports both the `stdio` and `streamable-http` variants in `mcp.json`.
6. If the host launches plugin subprocesses (i.e., stdio MCP servers), provides `PLUGIN_ROOT` and `PLUGIN_DATA` and expands both variables in runtime configuration values (`args`, `env`, `cwd`).
7. For stdio MCP servers, resolves `command` as a single executable token and uses the plugin root as the default subprocess working directory.
8. Supports at least one core component type (skills or MCP servers).

Client-specific behavior MUST be represented under `.<client>/`, not in root `plugin.json`. This preserves one strict portable manifest while allowing clients to experiment independently.

### 11.2 Incremental adoption

A host is not required to support every core component type. For example, a skills-only host can conform without supporting MCP servers, provided it satisfies all applicable requirements.

### 11.3 Unsupported components and failures

1. Hosts MUST ignore unsupported component types.
2. An invalid `plugin.json` is fatal to the plugin. As required by §5, the host MUST reject the plugin and MUST NOT discover or execute any of its components.
3. A failure isolated to a component type, component entry, or component process MUST NOT prevent the host from loading independently valid components. Hosts MUST apply the failure behavior defined for that component in §6 and §7.
4. Hosts SHOULD report invalid configuration and component failures. Hosts MAY report partially unsupported plugins, but lack of support for a component type is not itself an error.

---

## Appendix A: Conformance Checklist

*This checklist is for convenience only — when it conflicts with the spec text above, the spec governs.*

### Plugin loader

- [ ] Parse and validate `plugin.json` ([§5.1](#51-location-and-loading), [§5.2](#52-manifest-object))
- [ ] Validate required `id` and `name` fields ([§5.3](#53-required-fields))
- [ ] Validate plugin name against naming constraints ([§5.5](#55-plugin-name-constraints))
- [ ] Reject unknown `plugin.json` fields ([§5.2](#52-manifest-object))
- [ ] Reject package paths that resolve outside the plugin root ([§4.1](#41-general-requirements))
- [ ] Ignore unsupported `.<client>/` directories ([§8](#8-client-extensions))

### Component discovery

- [ ] Scan the fixed location for each supported component type ([§6.1](#61-fixed-locations))
- [ ] Ignore missing fixed locations without error ([§6.2](#62-missing-locations))

### MCP configuration

- [ ] Validate the closed `mcp.json` schema and each server variant ([§7.2.1](#721-discovery-and-configuration))
- [ ] If supporting MCP, implement both stdio and Streamable HTTP ([§7.2.1](#transport-support))
- [ ] Treat legacy HTTP+SSE as optional and never as an automatic fallback ([§7.2.1](#transport-support))
- [ ] Enforce remote URL and literal-header requirements ([§7.2.1](#streamable-http-and-legacy-httpsse))

### Environment and expansion

- [ ] If the host launches plugin subprocesses, provide `PLUGIN_ROOT` and a dedicated writable `PLUGIN_DATA` directory ([§9.1](#91-required-variables))
- [ ] Resolve MCP server `command` as a single bare or plugin-relative executable token ([§7.2.1](#721-discovery-and-configuration))
- [ ] Use the plugin root as the default MCP server working directory ([§7.2.1](#721-discovery-and-configuration))
- [ ] Expand `${PLUGIN_ROOT}` and `${PLUGIN_DATA}` in MCP server `args`, `env`, and `cwd` fields ([§9.2](#92-placeholder-expansion))

### Resilience

- [ ] Ignore unsupported component types ([§11.3](#113-unsupported-components-and-failures))
- [ ] Skip unsupported legacy HTTP+SSE servers without affecting other components ([§7.2.2](#722-loading-rules))
- [ ] Continue loading when an independent component fails ([§11.3](#113-unsupported-components-and-failures))
- [ ] Support at least one core component type ([§11.1](#111-minimum-host-requirements))

---

## Design Decisions

*This section explains why key design choices were made. It is for context only — the binding rules are in sections 1–11 above.*

### Why directory-based discovery?

Plugins use filesystem directories as the package unit rather than archive formats (`.zip`, `.tar.gz`) or registry-fetched bundles. This keeps plugins inspectable with standard tools (`ls`, `cat`, `git`), editable in-place during development, and compatible with version control without special tooling. Fixed root-level locations such as `skills/` and `mcp.json` eliminate discovery indirection, alternate-source precedence, and manifest configuration that every host would otherwise need to implement.

### Why only Agent Skills and MCP in v1?

Agent Skills and MCP have independently maintained formats with meaningful cross-host adoption. Other proposed component types — such as commands, hooks, agents, rules, and LSP servers — remain too host-specific for a stable portable contract and are outside portable v1 until their formats converge.

### Why root-level `plugin.json` is the conformance floor

Every conformant host MUST check `plugin.json` at the plugin root ([§5.1](#51-location-and-loading)). This gives plugin authors a single guaranteed manifest that works across all hosts without client-specific path knowledge. Client-specific behavior belongs under `.<client>/`, leaving the root manifest portable.

### Why a closed portable manifest?

Restricting root `plugin.json` to known portable fields enables strict validation, typo detection, and schema-driven key completion. It also prevents client experiments from claiming top-level names that a future Open Plugin version may need.

### Why dot-prefixed client directories?

The dot prefix creates a visible namespace boundary: unprefixed standard locations belong to Open Plugin, while `.<client>/` belongs to one client. Keeping client-specific capabilities outside `plugin.json` preserves the closed schema and prevents client-specific top-level component directories from colliding with future portable component types.

### Why an explicit MCP configuration format?

Existing hosts use incompatible MCP configuration shapes and infer transports differently. Open Plugin therefore defines an explicit closed union whose meaning is independent of any host-native format. Distinguishing Streamable HTTP from legacy HTTP+SSE also prevents an unexpected fallback to the deprecated transport.

### Why plugin variables over relative paths in configs?

MCP server arguments often need absolute paths at runtime. `${PLUGIN_ROOT}` provides an unambiguous, host-resolved anchor for bundled files, while `${PLUGIN_DATA}` identifies host-managed writable state that persists when package contents are replaced during an update. The `command` field does not use interpolation: a `./` path is resolved directly against the plugin root, and a bare name uses the platform's executable search rules. Treating `command` as one token avoids requiring hosts to parse and escape user-authored shell command strings.

### Why component failures are non-fatal

When an MCP server fails to start or connect, the host continues loading the plugin's remaining components ([§11.3](#113-unsupported-components-and-failures)). A plugin that provides skills and an MCP server should not become entirely unusable because one server is unavailable. The spec pairs non-fatal component failures with diagnostic requirements so that failures are visible rather than silent.

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

v1.0.0 defines failure-reporting requirements but does not standardize diagnostic or lifecycle event schemas. A future version may define:

- A standard event schema for plugin install, enable, disable, update, and uninstall actions
- Recommended fields: timestamp, actor (user or automation), plugin name, plugin version, action, outcome
- Integration points for forwarding audit events to external logging or SIEM systems
- Retention and access policies for audit records

### Dependency resolution

Plugins currently cannot declare dependencies on other plugins. A future version may define:

- A `dependencies` manifest field with version constraints
- Resolution order and conflict handling for transitive dependencies
- Peer dependency semantics for shared components

### Plugin testing and validation

No test harness or validation tool is specified. A future version may define:

- A `test` manifest field or convention
- A standard plugin linter or validator command
- Conformance test suites for host implementations
