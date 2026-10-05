# Trayector Implementation

This career profile follows **Trayector v0.1**, an open implementation of **Career as Code**.

- Specification: https://github.com/demonccc/trayector
- Machine-readable entry point: [`profile.json`](profile.json)
- Profile-owned vocabulary: [`settings.yaml`](settings.yaml)
- Synced AI/profile instructions: [`.trayector/README.md`](.trayector/README.md)

The repository README is intentionally a human-facing profile index. Trayector-specific modeling and generation instructions live in the managed `.trayector/` Profile Kit.

The canonical career knowledge lives in the linked profile, experience, project, deep-dive, story, feedback and content documents. `README.md`, tailored résumés and PDFs are derived views of that knowledge.

## Updating Trayector instructions

The repository includes [`.github/workflows/update-trayector.yml`](.github/workflows/update-trayector.yml).

It checks `demonccc/trayector@main` for changes to the upstream `profile-kit/`. When the managed instructions change, the workflow updates `.trayector/` on a dedicated branch and opens a pull request for review instead of changing the profile silently.

AI agents working with this repository should start with `profile.json`, then follow the `trayector.instructions` path before generating or modifying derived views.
