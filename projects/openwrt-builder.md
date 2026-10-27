---
id: project-openwrt-builder
type: project
category: open-source
period: 2026-present
status: active
capabilities:
  - embedded-systems
  - linux
  - firmware
  - build-automation
  - docker
  - python
  - ci-cd
---

# OpenWrt Builder

## Overview & Motivation

A reusable Python- and Docker-based workflow for creating customized OpenWrt firmware from versioned device profiles. It grew out of maintaining several OpenWrt routers and repeatedly adapting the TP-Link Archer A9 v6 QCN5502/ath9k patches for new releases.

## Architecture & Implementation

Profiles separate device and package configuration from the build process. The builder supports `imagebuilder`, `release-patched`, `selective-source`, and `full-source` strategies, with local Docker execution and GitHub Actions support. For the Archer A9 v6, the release-patched workflow combines patched kernel components with the exact official OpenWrt 25.12.6 SDK and ImageBuilder.

## My Contribution

I developed OpenWrt Builder, structured its build strategies and declarative profiles, and adapted the existing QCN5502/ath9k hardware support for the Archer A9 v6 in OpenWrt 25.12.5 and 25.12.6. Original QCN550x patch authors and related community work are explicitly credited in the accompanying article.

## Evidence

- Source: https://github.com/demonccc/openwrt-builder
- Device profile: https://github.com/demonccc/openwrt-builder/tree/main/profiles/tplink-archer_a9_v6-25.12.6
- Patched OpenWrt fork: https://github.com/demonccc/openwrt
- Firmware release: https://github.com/demonccc/openwrt-builder/releases/tag/tplink-archer_a9_v6-25.12.6-2026.10.26-1603-23
- [Technical article (English)](../content/2026-10-27-openwrt-builder/content.eng.md)
- [Artículo técnico (español)](../content/2026-10-27-openwrt-builder/content.esp.md)

## Tech Stack

OpenWrt, Python, Docker, GitHub Actions, ImageBuilder, SDK, Linux kernel, `ath9k`, device trees and cross-compilation.
