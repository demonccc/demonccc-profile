# Trayector Implementation

This career profile follows **Trayector v0.1**, an open implementation of **Career as Code**.

- Specification: https://github.com/demonccc/trayector
- Machine-readable entry point: [`profile.json`](profile.json)
- Profile-owned vocabulary: [`settings.yaml`](settings.yaml)
- Local AI/profile instructions: [`.trayector/README.md`](.trayector/README.md)
- Local Profile Kit version: [`.trayector/VERSION`](.trayector/VERSION)
- Trayector release baseline: [`.trayector/REF`](.trayector/REF)

The repository README is intentionally a human-facing profile index.

The canonical career knowledge lives in the linked profile, experience, project, deep-dive, story, feedback and content documents. `README.md`, tailored résumés and PDFs are derived views of that knowledge.

## Local Trayector instructions

`.trayector/` contains the local Trayector Profile Kit used by AI agents and profile tooling.

Those files may be customized for this profile. Local instructions are therefore not disposable generated files and must not be overwritten blindly by a framework update.

`VERSION` records the semantic Profile Kit version. `REF` records the immutable Trayector release tag used as the upstream baseline. This profile currently targets `v0.1.0` / `0.1.0`.

AI agents working with this repository should start with `profile.json`, then follow the `trayector.instructions` path before generating or modifying derived views.

## Updating Trayector instructions

The repository includes [`.github/workflows/update-trayector.yml`](.github/workflows/update-trayector.yml).

The update is manual: the repository owner chooses when to run it and explicitly selects a Trayector release tag. There is no scheduled update and updates do not follow `main`.

The workflow compares the previously accepted release from `.trayector/REF`, the current local `.trayector/` customizations, and the selected new Trayector release tag. It then opens a pull request with the proposed update.

Non-conflicting local changes are preserved. If the same instruction changed both locally and upstream and cannot be reconciled safely, the local file is kept, the incoming candidate is attached to the update PR under `.trayector/.update/`, and the conflict must be resolved manually before merge.
