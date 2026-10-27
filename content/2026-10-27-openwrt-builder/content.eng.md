---
classification: article
status: draft
topics:
  - openwrt
  - embedded-linux
  - firmware
  - build-automation
  - open-source
related_projects:
  - ../../projects/openwrt-builder.md
  - ../../projects/openwrt-wireless-mesh.md
publications: []
---

# From a Personal Need to a Community Tool: OpenWrt Builder

*OpenWrt Builder makes it possible to generate customized firmware through different build strategies, optimizing both time and resources. In this article, I explain how the tool works and how I used it to enable both Wi-Fi radios on the TP-Link Archer A9 v6.*

**Author:** [demonccc](https://github.com/demonccc)  
**Language:** English  
**Related project:** [OpenWrt Builder](https://github.com/demonccc/openwrt-builder)  

## How it all started

I've been using OpenWrt for a long time, both on my home network and in freelance projects for different clients. At home, I manage several routers with different architectures and update them regularly. Professionally, I've used it to implement connectivity and security solutions—such as VPNs, proxies, and traffic filtering—adapting networks without the limitations of vendor firmware.

However, every new version meant repeating the same manual process: preparing the build environment, selecting packages, applying source-code patches, and generating the final firmware image for each device.

The turning point was the TP-Link Archer A9 v6. Its 2.4 GHz Wi-Fi radio required specific patches to the `ath9k` driver, forcing me to repeat the integration manually with every update.

That's how OpenWrt Builder came about: a tool to automate these customized builds while separating each device's configuration from the different build methods.

## 1. OpenWrt Builder: separating configuration from the build process

OpenWrt Builder is not intended to replace the official tools. Instead, it lets you declare **what** you want to build and automates the process with Python inside Docker, both locally and through GitHub Actions.

Builds are defined through **profiles**: independent configurations specifying the OpenWrt version, packages, additional repositories, and device-specific parameters. Profile names follow the vendor–model–version convention; the build method is defined *inside* the profile, not in its name.

For example:

```text
profiles/
└── <vendor>-<model>-<openwrt_version>/
    ├── README.md
    ├── settings
    ├── packages
    ├── feeds
    ├── git-packages
    └── source-build-targets  (release-patched only)
```

Following this structure, the TP-Link Archer A9 v6 profile for OpenWrt 25.12.6 lives in `profiles/tplink-archer_a9_v6-25.12.6/`.

The Builder handles Docker environments, dependencies, and final artifacts, while each profile describes what a device needs. Adding another device or version only requires adding its configuration.

### Different needs, different build strategies

Not every customization requires the same computing effort. Adding LuCI or a couple of packages does not justify rebuilding an entire kernel; changing a hardware driver, on the other hand, requires working with the source code.

To address these different needs, OpenWrt Builder provides four build modes:

| Mode | What it does | Main advantage | Limitation |
| --- | --- | --- | --- |
| `imagebuilder` | Assembles an image from official binary packages without compiling source code. | Very fast; builds firmware locally or in GitHub Actions using declarative configuration in Git. | Cannot include changes to source code, the kernel, or drivers. |
| `release-patched` | Starts from an exact release, recompiles components affected by patches, and assembles the image with ImageBuilder. | Reuses official binaries and limits compilation to what is necessary. | Requires strict compatibility with the base version and its modules. |
| `selective-source` | Compiles firmware, selected packages, and their dependencies from source. | Supports development branches and source customization without building the entire package catalog. | May require building the kernel and toolchain; takes longer. |
| `full-source` | Enables a broad build of the OpenWrt source tree and its available packages. | Maximum coverage for package development and testing. | Higher build times and resource use, with potential failures unrelated to the target firmware. |

Choosing the right strategy improves build time, resource consumption, and maintainability.

### ImageBuilder: customizing firmware without compiling

This is the fastest method. It uses the official OpenWrt ImageBuilder for each architecture to assemble images from precompiled binary packages, without processing source code.

The entire software selection is versioned in Git as part of the profile, eliminating the need for external web-based builders or manually configured environments.

**Advantages**

- **Resource efficiency:** skipping toolchain, kernel, and library compilation dramatically reduces execution time.
- **Autonomy:** produces consistent images both locally and in CI/CD workflows.
- **Maintenance:** moving to another OpenWrt version usually involves updating the profile reference and checking package compatibility.

**Disadvantages**

- Limited to officially available binary packages and drivers.
- Cannot apply patches to the kernel or existing drivers.

**Recommended use:** officially supported devices where only the package selection or initial configuration files need customization.

### Release-patched: changing only what is necessary

This strategy is intended for situations where a stable firmware release works correctly but needs specific fixes in drivers or individual modules.

It combines the official SDK and ImageBuilder with a patched source tree. The SDK may also compile the required dependencies, but **only the explicitly declared custom packages are included in the final image**; everything else comes from the exact official release.

**Advantages**

- **Targeted compilation:** limits work to modified components and their necessary dependencies.
- **Stability:** preserves strict compatibility with the official base release.

**Disadvantages**

- Patches must be compatible with the exact SDK version.

**Recommended use:** applying specific fixes or adding hardware support to a stable release without rebuilding the entire system.

### Selective-source: building from source without compiling the whole catalog

This mode is useful when working with development branches or when patches modify structural parts of the system beyond what the SDK can handle. It builds the firmware and the declared package selection from source, generating the necessary tools and kernel while excluding the rest of OpenWrt's package catalog.

**Advantages**

- Flexibility to incorporate deep changes to the base system or follow active development branches.
- Precise control over source code and build options.

**Disadvantages**

- Longer execution time and higher resource consumption.
- Requires more attention to possible dependency conflicts.

**Recommended use:** active development, advanced customizations, or testing against development versions (snapshots).

### Full-source: building from the complete source tree

This mode provides full access to OpenWrt's build toolchain, enabling broad compilation of the source tree and the available package catalog.

**Advantages**

- Greater control over the build process and available packages.
- Useful for large-scale software validation, audits, or maintaining custom package repositories.

**Disadvantages**

- Substantial CPU time and storage requirements.
- Optional packages unrelated to the target device may fail during compilation.

**Recommended use:** repository maintenance, research, or comprehensive OpenWrt development.

## 2. Archer A9 v6: enabling both Wi-Fi radios

The TP-Link Archer A9 v6 uses the `ath79` platform and has two radios: a Qualcomm **QCN5502** (2.4 GHz, controlled by `ath9k`) and a Qualcomm **QCA9984** (5 GHz, handled by `ath10k`).

In official OpenWrt 25.12 builds, the 2.4 GHz radio remains disabled in the **Device Tree Source (DTS)**—the file describing the hardware to the kernel—because the official driver does not yet support QCN5502. Changing its status to `okay` is not enough: `ath9k` also needs to recognize the chip and correctly use ART calibration data.

### Adapting existing support

Building on work by **Wenli Looi**, **hdfla**, and **spike0en**, I adapted the patches for OpenWrt **25.12.5 and 25.12.6**, adjusting ART calibration and `ath9k` driver compatibility. The firmware is built using `release-patched`.

The changes are available in [my OpenWrt fork](https://github.com/demonccc/openwrt), particularly the `openwrt-25.12.6-archerA9v6` branch.

At the DTS level, the wireless interface is defined as follows:

```dts
&wmac {
    compatible = "qca,qcn5502-wifi";
    status = "okay";
    nvmem-cells = <&cal_art_1000>, <&macaddr_info_8 0>;
    nvmem-cell-names = "calibration", "mac-address";
};
```

### Building the firmware with OpenWrt Builder

For this device, the configuration profile uses the `release-patched` strategy. The official SDK compiles the modified Wi-Fi modules, while the Builder assembles the image using the adapted DTS on top of the stable base release:

```ini
METHOD=source
BUILD_MODE=release-patched
REPOSITORY=https://github.com/demonccc/openwrt.git
REF=openwrt-25.12.6-archerA9v6
BASE_REF=v25.12.6
TARGET=ath79
SUBTARGET=generic
DEVICE=tplink_archer-a9-v6
SDK=auto
```

`REF` identifies the branch of my fork containing the Archer patches, while `BASE_REF=v25.12.6` pins the official release used as the basis for the SDK, ImageBuilder, and unchanged components.

The `packages` file lists software by package name, and OpenWrt resolves the corresponding feeds and dependencies. The `feeds` file is required even when it is empty, indicating that no additional repositories are needed. `FEED_NAMES` is an optional setting that limits feed updates in supported modes.

In `release-patched`, `source-build-targets` identifies which components are rebuilt locally; dependencies compiled by the SDK are not automatically included as customized packages in the firmware.

To reproduce the build on Linux using Docker:

```bash
git clone https://github.com/demonccc/openwrt-builder.git
cd openwrt-builder

docker run --rm \
  --user "$(id -u):$(id -g)" \
  -e HOME=/tmp \
  -v "$PWD:/workspace" \
  demonccc/openwrt-builder:latest \
  python3 scripts/build.py build \
  --profile tplink-archer_a9_v6-25.12.6 \
  --output artifact \
  --cache-dir .cache/openwrt-builder
```

The process can run inside Docker, either locally or through GitHub Actions. When complete, it produces firmware to install on the Archer A9 v6 and test both Wi-Fi radios, along with a `BUILD_INFO` file distinguishing official components from those compiled locally. The OpenWrt 25.12.6 firmware release is linked below.

## 3. Two complementary projects

The combination of OpenWrt Builder and the Archer A9 v6 use case shows how automation and modularity simplify complex hardware support in embedded systems. Decoupling device definitions from the build environment makes it possible to maintain customized images without repeating the same manual tasks.

The main reason for sharing these projects is to offer the community a documented and reproducible method for generating custom images, adapting profiles to other routers, and continuing to improve hardware support in OpenWrt.

## Repositories and references

**Published projects**

- **OpenWrt Builder:** https://github.com/demonccc/openwrt-builder
- **Archer A9 v6 profile:** https://github.com/demonccc/openwrt-builder/tree/main/profiles/tplink-archer_a9_v6-25.12.6
- **OpenWrt fork with QCN5502 support:** https://github.com/demonccc/openwrt (version-specific branches for 25.12.5 and 25.12.6)
- **Archer A9 v6 firmware, OpenWrt 25.12.6:** https://github.com/demonccc/openwrt-builder/releases/tag/tplink-archer_a9_v6-25.12.6-2026.10.26-1603-23

**Reference work and credits**

- **OpenWrt:** https://github.com/openwrt/openwrt
- **Wenli Looi:** [GitHub profile](https://github.com/looi) · [Original QCN550x `ath9k` patch proposal, OpenWrt PR #9389](https://github.com/openwrt/openwrt/pull/9389)
- **hdfla/openwrt-qcn550x:** https://github.com/hdfla/openwrt-qcn550x
- **hdfla/openwrt-actions-qcn550x:** https://github.com/hdfla/openwrt-actions-qcn550x
- **spike0en/openwrt:** https://github.com/spike0en/openwrt
- **OpenWrt SDK and ImageBuilder:** https://github.com/openwrt/docker
