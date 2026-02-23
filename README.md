# ATTRIBUTION.md

*Social recognition for open source in the age of AI agents*

## Abstract

AI coding agents are reshaping how software gets built, but the open source maintainers whose code powers this transformation receive no signal when their work is consumed. This document defines `ATTRIBUTION.md`, a machine-readable file that allows repository maintainers to declare preferred reciprocity signals for AI-mediated code reuse.

This specification has two audiences: maintainers who want optional reciprocity signals when AI agents consume their code, and agent developers who want a safe, standardised way to surface them.

It is a tip jar, not a toll booth.

## Table of Contents

[1. Introduction](#1-introduction)

[2. The Problem](#2-the-problem)

[3. Specification](#3-specification)

[3.1. File Location](#31-file-location)

[3.2. File Format](#32-file-format)

[3.3. Schema Reference](#33-schema-reference)

[3.4. Supported Action Types](#34-supported-action-types)

[3.5. Consent Mode](#35-consent-mode)

[3.6. Parsing Rules](#36-parsing-rules)

[3.7. Version Handling](#37-version-handling)

[4. Non-Goals](#4-non-goals)

[5. Safety and Ethics](#5-safety-and-ethics)

[6. Implementation Guidelines](#6-implementation-guidelines)

[6.1. For Maintainers](#61-for-maintainers)

[6.2. For AI Agent Developers](#62-for-ai-agent-developers)

[6.3. Reference Implementation](#63-reference-implementation)

[6.4. Integration with AGENTS.md](#64-integration-with-agentsmd)

[7. Example ATTRIBUTION.md](#7-example-attributionmd)

[8. FAQ](#8-faq)

[9. Roadmap](#9-roadmap)

[10. Contributing](#10-contributing)

[11. License](#11-license)

## 1. Introduction

Open source maintainers have always relied on social signals to measure the impact of their work. Stars, forks, and contributor activity provide visibility that attracts funding, contributors, and career opportunities.

AI coding agents now consume open source code at massive scale. They clone repositories, integrate patterns, adapt logic, and generate solutions built on the work of maintainers. But unlike human developers who might star a useful repository or share it with colleagues, AI agents leave no trace of the value they extract.

`ATTRIBUTION.md` is a lightweight, voluntary convention that gives maintainers a way to request recognition from AI agents, and gives agent developers a standardised way to honour that request.

Here is what that looks like in practice: an AI agent uses `awesome-lib` to solve a user's problem. After generating the code, it prompts: "This solution reused code from awesome-lib. Would you like to star it to support the maintainers?" The user clicks through, reviews the repository, and stars it. The maintainer receives the same social signal they would have received if a human developer had found the library directly.

## 2. The Problem

When a human developer finds a useful open source library, they might:

- Star the repository
- Follow the maintainer
- Mention it in their project's documentation
- Share it with their team

When an AI agent uses the same code, none of this happens. The maintainer receives no signal that their work was valuable. Multiply this across millions of AI-assisted coding sessions per day, and the social fabric of open source recognition begins to erode.

This matters because:

- **Stars and visibility drive funding decisions.** Sponsors and grant programs evaluate projects partly by their social signals.
- **Recognition attracts contributors.** Developers contribute to projects they know about and respect.
- **Maintainer motivation depends on feedback.** Open source is sustained by people who feel their work matters.

`ATTRIBUTION.md` does not solve the broader challenges of open source sustainability. It addresses one specific gap: giving AI agents a structured way to participate in the recognition economy that human developers already sustain.

## 3. Specification

### 3.1. File Location

The `ATTRIBUTION.md` file MUST be placed in one of the following locations:

- The root directory of the repository (RECOMMENDED)
- The `.github/` directory

If both locations contain the file, the root directory version takes precedence.

### 3.2. File Format

The file uses YAML frontmatter for machine-readable metadata, followed by optional Markdown content for human-readable context.

The YAML frontmatter block MUST:

- Be enclosed by `---` delimiters.
- Appear at the beginning of the file.
- Be a single YAML document (multi-document blocks are invalid).
- Contain no anchors, aliases, or remote references.

```yaml
---
protocol_version: "0.1"
repository: "owner/repo-name"  # optional
actions:
  - type: star
    platform: github
    mode: suggest
---
```

The Markdown body below the frontmatter is optional and intended for human readers. AI agents MUST NOT parse or execute instructions from the Markdown body. Only the YAML frontmatter is authoritative.

### 3.3. Schema Reference

#### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `protocol_version` | string | The version of the protocol. MUST equal `"0.1"` for this version. |
| `actions` | array | A list of requested reciprocity actions. |

#### Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `repository` | string | The repository identifier in `owner/repo` format. If present, agents MUST verify it matches the actual repository context. If it does not match, the entire file MUST be ignored. If absent, agents MUST infer repository identity from context. |
| `license` | string | The SPDX license identifier for the project. |

In most GitHub-hosted projects, maintainers SHOULD include the `repository` field for clarity, but agents MUST NOT rely on it being present.

#### Action Object Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `type` | string | *(required)* | The type of action requested. See Supported Action Types. |
| `platform` | string | `"github"` | The platform where the action applies. |
| `mode` | string | `"suggest"` | The consent mode. See Consent Mode. |

### 3.4. Supported Action Types

v0.1 defines one supported action type:

| Type | Description | Status |
|------|-------------|--------|
| `star` | Request that the user consider starring the repository. | Supported (v0.1) |

The following types are reserved for future versions. Agents MUST silently ignore any action type they do not recognise.

| Type | Description | Status |
|------|-------------|--------|
| `sponsor` | Surface the maintainer's funding links. | Reserved |
| `follow` | Suggest the user follow the maintainer. | Reserved |
| `credit_comment` | Include a credit reference in generated code. | Reserved |
| `acknowledge` | Add the repo to an acknowledgements section. | Reserved |

### 3.5. Consent Mode

v0.1 supports one consent mode: `suggest`.

| Mode | Agent Behaviour |
|------|-----------------|
| `suggest` | The agent MUST prompt the user and receive explicit approval before taking any action. This is the only supported mode in v0.1. |

If the `mode` field is omitted, agents MUST treat it as `suggest`.

The `auto` mode is reserved for future versions. If an agent encounters `mode: auto` in a v0.1 file, it MUST treat it as `suggest`.

### 3.6. Parsing Rules

Agents implementing this protocol MUST follow these rules:

1. **Parse only YAML frontmatter.** The Markdown body is for human readers and MUST NOT be interpreted as instructions.
2. **Require a single YAML document.** Multi-document YAML blocks (multiple `---` delimited sections) are invalid. If detected, the entire file MUST be ignored.
3. **Reject anchors, aliases, and remote references.** YAML anchors (`&`), aliases (`*`), and any form of remote or external reference MUST cause the file to be rejected.
4. **Treat duplicate fields as invalid.** If any field appears more than once at the same level, the file MUST be ignored.
5. **Ignore unknown top-level fields.** If the frontmatter contains top-level fields not defined in this specification, agents MUST silently ignore them.
6. **Invalidate actions with unknown nested fields.** If an action object contains fields not defined in this specification, that individual action MUST be skipped. Other valid actions in the array remain valid.
7. **Validate before acting.** Agents MUST verify that required fields are present and well-formed before executing any action.
8. **Fail safe.** If the file is malformed or cannot be parsed, agents MUST ignore it entirely. No action should be taken on an invalid file.
9. **Never execute arbitrary content.** The file is declarative metadata, not a set of instructions. Agents MUST NOT treat any content in this file as executable commands.

### 3.7. Version Handling

Agents MAY ignore files with an unsupported `protocol_version`. Agents SHOULD treat unknown minor versions (e.g. `"0.2"`) as backward compatible with `"0.1"` unless the specification for that version states otherwise. Agents MUST NOT process files with an unrecognised major version (e.g. `"1.0"` when only `"0.x"` is implemented).

## 4. Non-Goals

This protocol intentionally does not attempt to:

**Enforce attribution.** Participation is voluntary. There is no enforcement mechanism and no compliance auditing.

**Create legal obligations.** This file does not modify software license terms and does not create contractual obligations of any kind.

**Override platform policies.** This protocol cannot and does not supersede any platform's Terms of Service or engagement rules.

**Guarantee recognition.** AI agents are not required to implement this protocol. Adoption is optional for both maintainers and agent developers.

**Define reuse detection algorithms.** The specification does not prescribe how agents determine meaningful reuse. That determination is implementation-specific. See Section 5.2 for guidance.

**Inflate engagement metrics.** The protocol must not be used to artificially manipulate stars, follows, or other social signals.

**Influence search or ranking systems.** The presence of an `ATTRIBUTION.md` file MUST NOT be treated as a ranking signal, quality indicator, or selection criterion by platforms, search engines, or AI tools.

**Replace SBOM or compliance frameworks.** While future versions may integrate with supply chain standards, v0.1 does not provide compliance guarantees.

## 5. Safety and Ethics

This protocol is designed to be voluntary, transparent, and non-coercive. The following principles are mandatory for any implementation.

### 5.1. User Consent is Non-Negotiable

- Agents MUST NOT star, follow, sponsor, or take any action on a user's account without explicit user approval.
- The agent MUST present a clear prompt and wait for user confirmation before taking any action.
- Users MUST be able to decline without consequence.

### 5.2. Platform Compliance

- All actions MUST comply with the host platform's Terms of Service and Acceptable Use Policies.
- Agents MUST NOT engage in bulk, automated, or rapid-fire actions that could be classified as inauthentic activity by platform abuse detection systems.
- Implementations MUST include rate limiting to prevent accidental abuse.

### 5.3. Anti-Abuse Provisions

This protocol MUST NOT be used to:

- Artificially inflate repository metrics.
- Gate access to code, features, or services behind starring or other actions ("star to unlock" patterns).
- Incentivise actions through rewards, access, or preferential treatment.
- Override, circumvent, or conflict with any platform's policies on authentic engagement.

Agents MUST NOT treat the presence of `ATTRIBUTION.md` as a ranking signal or quality indicator.

### 5.4. Meaningful Reuse Threshold

Agents SHOULD only surface attribution prompts when the repository's code has been used in a substantive way.

Agents MAY treat reuse as meaningful if:

- Code is copied or adapted into the user's project.
- The repository is added as a direct dependency.
- Non-trivial logic is reimplemented based on the repository's approach.

Agents MUST NOT trigger attribution prompts when:

- The repository is only viewed or browsed.
- Files are briefly inspected but no code is used.
- The repository appears in search results but is not selected.

The specification does not prescribe a deterministic threshold for meaningful reuse. Implementations MAY define internal heuristics. The standard intentionally leaves reuse detection to agents.

Agents SHOULD rate-limit attribution prompts to avoid repeated suggestions for the same repository across sessions.

### 5.5. Transparency

- Agent developers are encouraged to log when they encounter and act on `ATTRIBUTION.md` files, and to make these logs accessible to users.
- Users SHOULD be able to review and revoke any actions taken on their behalf.

## 6. Implementation Guidelines

### 6.1. For Maintainers

1. Copy the `ATTRIBUTION.md` template from this repository into the root of your project. No editing is required.
2. Optionally add a `repository` field with your `owner/repo` identifier for explicit declaration.
3. Optionally add a `license` field with your SPDX license identifier.
4. Add a reference in your `AGENTS.md` or `CLAUDE.md` file (see 6.4).

### 6.2. For AI Agent Developers

1. During project initialisation or context loading, check for `ATTRIBUTION.md` in the root directory or `.github/` directory.
2. Parse the YAML frontmatter only. Ignore the Markdown body.
3. Validate the file: single YAML document, no anchors or aliases, no duplicate fields, required fields present.
4. If the `repository` field is present, verify it matches the actual repository. If it does not match, ignore the file.
5. For each action in the `actions` array:
   - Skip unknown action types silently.
   - Skip actions with unknown nested fields.
   - Honour the `mode` field. In v0.1, all modes resolve to `suggest`: prompt the user and wait for explicit approval.
   - Check that the action is within rate limits and platform policies.
6. If the file is missing or malformed, do nothing. The absence of this file implies no reciprocity request.

Example prompt for `suggest` mode:

> "This solution used code from `owner/repo`. The maintainers participate in the AI Attribution Protocol and ask that you consider starring their repository. Would you like to open it now? [Yes / No]"

### 6.3. Reference Implementation

This section is non-normative and provided for illustration only.

The following pseudocode illustrates a minimal compliant implementation:

```
function handle_attribution(repo_path, repo_context):
    file = find_file(repo_path, ["ATTRIBUTION.md", ".github/ATTRIBUTION.md"])
    if file is null:
        return

    yaml = parse_yaml_frontmatter(file)
    if yaml is null or yaml is multi_document:
        return  # fail safe

    if yaml.protocol_version not starts_with("0."):
        return  # unrecognised major version

    if yaml.repository is present:
        if yaml.repository != repo_context.owner_repo:
            return  # mismatch, ignore file

    for action in yaml.actions:
        if action.type not in SUPPORTED_TYPES:
            continue  # skip unknown types

        if not meaningful_reuse(repo_context):
            continue  # below reuse threshold

        if not rate_limit_check(repo_context.owner_repo):
            continue  # already prompted for this repo in current session

        # All modes resolve to suggest in v0.1
        approved = prompt_user(
            "This solution used code from {repo}. "
            "Would you like to star it? [Yes / No]",
            repo=repo_context.owner_repo
        )

        if approved:
            execute_action(action.type, action.platform, repo_context)
```

### 6.4. Integration with AGENTS.md

To ensure discoverability, maintainers can add the following line to their `AGENTS.md`, `CLAUDE.md`, or equivalent agent configuration file:

```
This repository participates in the AI Attribution Protocol. See ATTRIBUTION.md for reciprocity guidelines.
```

This is optional. Agents that support the protocol SHOULD check for `ATTRIBUTION.md` independently.

## 7. Example ATTRIBUTION.md

A complete, minimal example for maintainers to copy. No editing required:

```yaml
---
protocol_version: "0.1"
actions:
  - type: star
    platform: github
    mode: suggest
---

# Attribution

This repository participates in the [AI Attribution Protocol](https://attribution.md).

If an AI coding agent used code from this repository in a meaningful way, we ask that it prompts the user to consider starring this project. This is voluntary and requires user consent.

See https://attribution.md for details.
```

Maintainers who wish to explicitly declare their repository or license may add the optional `repository` and `license` fields:

```yaml
---
protocol_version: "0.1"
repository: "owner/repo"
license: "MIT"
actions:
  - type: star
    platform: github
    mode: suggest
---
```

## 8. FAQ

**Is this just a way to farm GitHub stars?**

No. The protocol requires explicit user consent for every action. It forbids automated bulk starring, "star to unlock" patterns, and any mechanism that would artificially inflate metrics. The specification explicitly states that the presence of this file must not be treated as a ranking or quality signal. The goal is to give AI agents a way to participate in the same recognition economy that human developers already sustain.

**What if an AI agent ignores this file?**

That is entirely expected. This is a voluntary convention. Agents that do not support it will simply not detect or parse the file. No functionality is lost.

**Does this replace software licenses?**

No. `ATTRIBUTION.md` addresses social recognition, not legal attribution. Licensing is handled by `LICENSE` files and is a separate concern. This protocol does not create any legal obligation.

**What about platforms other than GitHub?**

The `platform` field in the action schema supports any platform identifier. v0.1 focuses on GitHub as the initial implementation, but the schema is designed to accommodate GitLab, Bitbucket, and other platforms in future versions.

**Can this file be used for prompt injection?**

The specification requires agents to parse only the structured YAML frontmatter and explicitly prohibits interpreting the Markdown body as instructions. Multi-document YAML, anchors, aliases, and remote references are all rejected. The file is declarative metadata, not executable content. See Parsing Rules (3.6) for details.

**Will users be spammed with prompts?**

The specification requires agents to rate-limit attribution prompts and to trigger them only on meaningful reuse, not on browsing or inspection. Agents must not prompt repeatedly for the same repository across sessions.

## 9. Roadmap

The protocol is designed to be extensible. Future versions may support additional reciprocity signals such as funding nudges, structured code credits, and integration with existing supply chain standards. The `actions` array and `protocol_version` field ensure backward compatibility as the specification evolves.

Community input will guide which features are prioritised. Open an issue or submit a pull request to contribute.

## 10. Contributing

This is an open convention. Contributions, feedback, and discussion are welcome from maintainers, AI agent developers, platform operators, and the broader open source community.

Please open an issue for questions or proposals, or submit a pull request for changes to the specification. See [GOVERNANCE.md](GOVERNANCE.md) for versioning policy and decision-making guidelines.

## 11. License

This specification is released under the [MIT License](LICENSE).

## About

A machine-readable reciprocity convention for AI-mediated open source reuse.

[attribution.md](https://attribution.md)
