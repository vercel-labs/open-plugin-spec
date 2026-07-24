# Governance

## Technical Charter

This Charter sets forth the responsibilities and procedures for technical contribution to, and oversight of, **Agent Plugins** (the “Project”), an open, vendor-neutral specification for packaging reusable components that extend AI agents into distributable plugins.

Agent Plugins is operated as a community-governed open specification project. All contributors (including those serving as Maintainers, Core Maintainers, the Lead Core Maintainer, and other technical roles) and all other participants in the Project (collectively, “Collaborators”) must comply with the terms of this Charter.

---

## 1. Mission and Scope

The mission of the Project is to define, maintain, and promote an open, vendor-neutral specification for packaging reusable components that extend AI agents into distributable plugins.

The scope of the Project includes:

- The Agent Plugins Specification
- Reference implementations
- Conformance tests
- Documentation
- Tooling and other artifacts that support interoperable clients, plugin packages, and ecosystems built on the specification

The Project exists to enable portability, competition, and long-term stability for plugin packages across vendors and platforms.

---

## 2. Technical Governance Structure

The Project adopts a hierarchical technical governance structure:

- **Contributors** – anyone who submits code, documentation, specifications, issues, or other technical artifacts
- **Maintainers** – Contributors with commit access to one or more Project repositories
- **Core Maintainers** – Maintainers responsible for the overall technical direction and integrity of the specification
- **Lead Core Maintainer** – one Core Maintainer designated as final technical decision-maker when consensus cannot be reached

All governance roles are held by individuals, not organizations. No seats are reserved for specific companies.

No single vendor may control a majority of Core Maintainer seats.

The **Technical Steering Committee (TSC)** consists of all Core Maintainers together with the Lead Core Maintainer. The TSC is responsible for all technical oversight of the Project.

The current list of Core Maintainers and the Lead Core Maintainer shall be recorded in the Project’s MAINTAINERS file. The TSC may define alternative methods for selecting or rotating voting members, provided such methods are publicly documented.

TSC meetings are open and may be conducted electronically or in person.

---

## 3. Roles and Promotion

Unless otherwise documented:

- Contributors include anyone who submits technical artifacts
- Maintainers are Contributors with commit rights
- Core Maintainers are Maintainers with responsibility for the specification, architecture, and cross-repository direction
- The Lead Core Maintainer serves as TSC Chair unless another Core Maintainer is designated

Promotion rules:

- Contributor → Maintainer: majority approval of the TSC
- Maintainer → Core Maintainer: majority approval of the TSC

Removal rules:

- Maintainer: majority approval of the TSC
- Core Maintainer: decision by the Lead Core Maintainer after consultation with the TSC

Participation in the Project is open to anyone who abides by this Charter and the Project’s policies.

---

## 4. Core Maintainer Nomination and Removal

Any Maintainer or Core Maintainer may nominate a candidate for Core Maintainer.

The Lead Core Maintainer evaluates nominations based on:

- technical contribution
- alignment with Project principles
- community conduct
- multi-vendor neutrality

The Lead Core Maintainer will publicly confirm or decline the nomination with stated reasoning.

Any Collaborator may raise a proposal to remove a Core Maintainer. The remaining Core Maintainers (excluding conflicted parties) will review and publish a decision.

---

## 5. Lead Core Maintainer Selection and Removal

The Lead Core Maintainer may be removed by a super-majority (75%) vote of all Core Maintainers, excluding the incumbent.

Upon removal, resignation, or permanent unavailability, the Core Maintainers will select a new Lead Core Maintainer using a consensus-seeking or ranked-choice voting method documented in the CONTRIBUTING file.

---

## 6. Responsibilities of the TSC

The TSC is responsible for:

- stewarding the Agent Plugins Specification
- approving changes, extensions, and deprecations
- managing reference implementations and test suites
- establishing release, compatibility, and versioning policies
- creating working groups
- appointing liaisons to other standards bodies or open-source projects
- defining contribution and review workflows
- resolving technical disputes
- coordinating communications about the Project

---

## 7. Voting

The Project seeks consensus. When votes are required:

- Each TSC member has one vote
- Quorum requires 50% of TSC members
- Decisions require a majority of those present unless otherwise specified
- Electronic votes require a majority of all TSC members

If the TSC cannot resolve a dispute, the Lead Core Maintainer will act as the tie-breaker; if the dispute remains unresolved, the TSC may convene a special subcommittee of TSC members to develop a recommendation for final consideration by the TSC.

---

## 8. Community Principles

The Project operates openly, transparently, and collaboratively.

No individual or organization may be excluded except for failure to follow documented rules applied fairly to all.

All proposals, decisions, and technical discussions shall be publicly accessible.

---

## 9. Intellectual Property

Contributors retain copyright in their contributions.

Unless otherwise approved by the TSC:

- Specification text and documentation are licensed under **Creative Commons Attribution 4.0 (CC-BY-4.0)**
- Code is licensed under the **Apache 2.0 License**

The TSC may approve alternative licenses for specific Project materials by a two-thirds vote.

---

## 10. Project Assets and Trademarks

The name **Agent Plugins**, associated logos, domains, and GitHub organizations are held in trust for the Project by a neutral entity designated by the TSC.

No single vendor may claim exclusive ownership or control of Project identity or infrastructure.

---

## 11. Amendments

This Charter may be amended by a two-thirds vote of the entire TSC.
