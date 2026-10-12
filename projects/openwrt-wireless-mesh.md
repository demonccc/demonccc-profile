---
id: project-openwrt-wireless-mesh
type: project
category: personal-lab
period: 2018-present
status: active
capabilities:
  - networking
  - linux
  - embedded-systems
  - firmware
  - wireless
---

# OpenWrt Wireless Mesh & Custom Firmware

## Overview & Motivation

A long-running home networking lab focused on building reliable wireless coverage, experimenting with open routing/mesh technologies and compiling custom OpenWrt firmware for real hardware.

## Architecture & Implementation

The network uses multiple OpenWrt routers and B.A.T.M.A.N. Advanced at layer 2, with roaming support based on 802.11r/k/v and related tooling. Custom firmware builds are produced from source when stock images do not provide the required modules or package combinations.

## My Contribution

I designed the topology, selected and flashed hardware, configured the mesh and roaming behavior, debugged driver and routing issues, and maintained custom build workflows for firmware and packages.

## Key Capabilities Demonstrated

Wireless networking, Linux networking internals, embedded firmware, cross-compilation, hardware compatibility analysis and hands-on troubleshooting.

## Evidence

Representative public repositories:

- https://github.com/demonccc/openwrt
- https://github.com/demonccc/openwrt-actions-qcn550x

## Tech Stack

OpenWrt, batman-adv, 802.11r/k/v, DAWN, Linux kernel, Shell, Makefiles and GitHub Actions.
