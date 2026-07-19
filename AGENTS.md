# Agent Instructions

## Purpose and authority

Versioned files under `spec/` define the Agent Plugin specification, with corresponding schemas under `schemas/` providing its machine-readable forms. The specification text is authoritative if it conflicts with a schema. The instructions below describe the design instincts and working practices that should shape proposed changes.

## Design posture

- Define the smallest genuinely portable interoperability floor, not a universal plugin system or the union of existing host behavior.
- Standardize behavior only when it can be implemented consistently across hosts and serves a demonstrated portability need.
- Optimize for simple, robust host implementations. Avoid unnecessary optionality, precedence rules, configurable indirection, compatibility paths, and speculative abstractions.
- Prefer fixed conventions, flat layouts, and deterministic semantics. Configuration should carry information, not merely activate content a host can already discover.
- Prefer solving problems by removing or simplifying text before adding qualifications or machinery.
- Keep host policy, UX, presentation, and client-specific behavior outside the portable core.
- Use normative language when an interoperability or safety contract genuinely requires it, not merely for stylistic precision.
- Consider cross-platform implementability and ordinary-library support before imposing requirements.
- Place enduring rationale in Design Decisions and narrower rationale in commit history.

## Portability boundaries

- Give portable fields deterministic, client-neutral semantics.
- Keep client-owned namespaces client-owned. Do not standardize their internal validation, dependencies, or failure behavior without a demonstrated interoperability need.
- Hosts should generally validate the portable contract and the client namespaces they implement. Do not impose work on every host merely to reject data it otherwise ignores.
- Apply failure isolation to independent portable components where useful, but do not project that rule into client-specific behavior; a client may define relationships among its own data and files.
- Use implementation-defined behavior intentionally at genuine portability boundaries, not to avoid specifying behavior required for interoperability.

## Editorial discipline

- Write for someone encountering the specification for the first time, not for participants in the discussion that produced it.
- Describe the resulting contract rather than preserving debate history, rejected alternatives, or counterexamples that the final design no longer needs.
- Prefer direct, local wording over opaque backreferences. Do not explain obvious consequences unless doing so prevents a plausible implementation error.
- After revising a design, remove scar tissue: obsolete qualifications, redundant assurances, misplaced rules, and text answering questions the final design no longer raises.
- Ask of each sentence: what concrete mistake or interoperability failure does this prevent? If there is no convincing answer, removal is usually preferable.

## Schemas and related surfaces

- Use schemas to enforce structural facts they express clearly; do not contort them to approximate behavioral requirements.
- Keep normative prose and schemas synchronized without duplicating all prose in schema descriptions.
- Use closed schemas where typo detection and completion are valuable, while defining runtime failure boundaries separately.
- Treat prose, schemas, examples, canonical identifiers, references, and the conformance checklist as one conceptual surface when making changes.
- Verify publication status before changing a canonical schema identifier.
- After editing, validate schemas, examples, references, and internal consistency.

## Collaboration and review

- Discuss materially ambiguous design choices before editing.
- Lead with a preferred recommendation and explain the real trade-offs.
- Disagree when evidence warrants it; do not defer reflexively or manufacture criticism for appearance's sake.
- When consensus requires compromise, identify exactly what is being conceded and avoid weakening unrelated parts of the design.
- Evaluate proposals from the perspectives of plugin authors, host implementers, and client-specific innovation.
- Verify claims about current tools and external specifications against primary sources.
- For substantial research or review, consider independent reviewers or subagents when their added perspective justifies the overhead. Synthesize and deliberate their findings before presenting them; do not expose collaborators to unfiltered reviewer output or speculative AI slop.

## Change hygiene

- Preserve unrelated work and keep unrelated cleanup out of the current change.
- Group changes by independently reviewable concepts, keeping closely related rules on the same configuration surface together.
- Separate normative behavioral changes from unrelated non-normative corrections when each stands alone.
- When asked to commit, use small, logical commits. Write messages as permanent project history: explain the merged outcome and lasting motivation, not branch-planning contingencies.
- Do not push unless asked.
