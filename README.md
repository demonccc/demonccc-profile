# Claudio Sánchez — Career Profile

A structured, versioned record of professional experience, technical knowledge, projects, decisions, ideas and evidence.

This repository is a **reference implementation of [Trayector](https://github.com/demonccc/trayector) v0.1**, an open implementation of **Career as Code**.

> A job title provides context. Evidence demonstrates capability.

## What lives here

This repository is not intended to be a résumé stored in Git.

It is the source of truth for a broader professional history: what I worked on, what I designed, what I built, what I led, what I learned, what I would do differently, and the evidence behind those claims.

A résumé, LinkedIn profile, portfolio or interview can be generated from or informed by this repository, but none of them is the repository itself.

## Navigate

- [`profile/summary.md`](profile/summary.md) — current professional summary
- [`profile/career-timeline.md`](profile/career-timeline.md) — chronological view of the career
- [`profile/capabilities.md`](profile/capabilities.md) — capabilities and their supporting evidence
- [`profile/education.md`](profile/education.md) — education and formal learning
- [`experience/`](experience/) — professional experience, split by organization and meaningful role/seniority changes
- [`deep-dives/`](deep-dives/) — detailed architecture and engineering case studies
- [`projects/`](projects/) — personal, open-source, lab and other projects
- [`stories/`](stories/) — cross-cutting career stories, decisions, failures and lessons learned
- [`content/`](content/) — canonical professional content using a single bundle format
- [`settings.yaml`](settings.yaml) — profile-owned language and classification vocabulary
- [`generated/`](generated/) — machine-generated indexes and derived artifacts

## Content convention

Every content item uses the same structure. Whether it is a LinkedIn post, an article, a note or another kind of professional content is expressed through metadata, not through different folders.

```text
content/
└── <date>-<slug>/
    ├── content.<language-code>.md
    └── assets/
        └── [optional files]
```

The language code and the available classifications are defined in [`settings.yaml`](settings.yaml), so both humans and AI systems can resolve their meaning by reading the repository itself.

## How this repository models a career

Trayector separates concepts that traditional résumés often collapse together:

```text
Role
  → provides context

Contribution
  → what I actually did

Capability
  → what that work demonstrates

Evidence
  → where the claim can be inspected

Outcome
  → what changed as a result
```

This matters because capability does not always match a formal title. A manager can design an architecture. An individual contributor can lead a transformation. A personal project can demonstrate knowledge that never appeared in a job description.

## Machine-readable entry point

[`profile.json`](profile.json) provides the entry point for tools, parsers and AI systems.

The human-readable Markdown remains canonical for the narrative content; structured metadata provides navigation and relationships without duplicating the entire career in JSON.

## Specification

This repository follows **Trayector v0.1**.

- [Trayector](https://github.com/demonccc/trayector)
- [Getting started](https://github.com/demonccc/trayector/blob/main/docs/getting-started.md)
- [Specification](https://github.com/demonccc/trayector/tree/main/spec)
- [Templates](https://github.com/demonccc/trayector/tree/main/templates)

Templates and schemas intentionally live in Trayector rather than being duplicated here.

## Status

The repository structure has been migrated to Trayector v0.1. Career content will be added progressively and used to validate and evolve the specification against a real, long-running technology career.
