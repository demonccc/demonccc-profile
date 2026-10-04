# Trayector

---
id: project-trayector
type: project
category: open-source
period: 2026-present
status: active
capabilities:
  - career-as-code
  - knowledge-modeling
  - information-architecture
  - specification-design
  - developer-tooling
  - open-source
---

## Overview & Motivation

Trayector is an open implementation of **Career as Code**: a structured, versioned way to represent professional knowledge beyond the limits of a résumé.

The project started from a practical problem: job titles and CV summaries lose too much context about what a person actually designed, built, led, learned and can prove. Trayector treats the career as inspectable knowledge that can be read directly by humans and AI systems.

## Architecture & Implementation

Trayector v0.1 is intentionally based on portable, low-friction formats: Markdown, YAML, JSON and Git.

The model separates concepts that traditional résumés usually collapse together:

- roles provide context;
- contributions describe what the person actually did;
- capabilities express what that work demonstrates;
- evidence makes claims inspectable;
- outcomes describe what changed.

The repository contains the specification, principles, JSON Schema, templates and examples needed to build compatible profiles. Human-readable Markdown remains canonical for narrative knowledge, while structured metadata provides stable identifiers, navigation and relationships.

## My Contribution

I conceived the Career as Code approach and created Trayector as its open implementation.

My work includes:

- defining the core concepts and semantics;
- designing the repository and document conventions;
- defining the relationship between experience, capabilities, contributions, evidence, projects, stories, deep dives and content;
- establishing the rule that information reliably derivable from a path or filename should not be duplicated in metadata;
- defining profile-owned vocabularies instead of imposing universal taxonomies where the profile owner should control meaning;
- designing the content-bundle convention for multilingual professional content;
- building `demonccc-profile` as the first real reference implementation and using real career material to validate and evolve the specification.

## Key Capabilities Demonstrated

Knowledge modeling, information architecture, specification design, developer tooling, open-source product design, human-readable data modeling and AI-readable professional knowledge representation.

## Results & Lessons

Trayector has reached a working v0.1 draft with a real reference implementation rather than only an abstract schema.

A central design lesson is that Career as Code must stay understandable without specialized tooling. If a human needs to understand Trayector internals before understanding a career document, the representation is too complicated.

Another important principle is that the specification should define structure and semantics without unnecessarily owning personal taxonomies that can remain explicit inside the profile itself.

## Evidence

- Trayector repository: https://github.com/demonccc/trayector
- Reference implementation: https://github.com/demonccc/demonccc-profile

## Tech Stack

Markdown, YAML, JSON, JSON Schema, Git and GitHub.
