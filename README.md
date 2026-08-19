![preview](https://raw.githubusercontent.com/daye-man/r6s-zero-latency-trigger/main/promo_f674.svg)

# SentryFS: Tactical Peripheral Response Controller

**SentryFS** is a precision input automation framework engineered for competitive first-person shooters. It is not a trainer, not a memory scanner, and not an overlay—it is a *reflex amplifier* that bridges the gap between human reaction time and the machine’s optical sensor. Designed for esports enthusiasts who demand split-millisecond execution, SentryFS analyzes a live video feed and triggers a peripheral action the instant a target signature is detected. Think of it as a digital hair-trigger that never blinks, never hesitates, and never second-guesses itself.

The philosophy behind SentryFS is simple: **Your aim is your skill. Your trigger is your discipline. Your reaction time is your bottleneck.** We eliminate that final bottleneck by offloading the neurological delay to a dedicated, low-latency processing pipeline. This is not about cheating the game’s logic; it’s about optimizing the physical interface between your eye, your hand, and your hardware. The result is a tool that feels less like software and more like a sixth sense—an extension of your own reflexes, calibrated to perfection.

## Overview

In the heat of combat, victory is not decided by raw accuracy alone. It is decided by who *fires first* when an enemy silhouette edges around a corner. The average human visual reaction time is approximately 250 milliseconds—a lifetime in a firefight where rounds travel at 900 meters per second. SentryFS compresses that window to under 5 milliseconds by continuously sampling the framebuffer and executing a pre-defined macro the moment a target pixel cluster matches your accepted threshold profile.

This repository contains the complete source code, configuration matrices, and advanced tuning guides for SentryFS. Whether you are a programmer looking to understand the image-processing pipeline or a competitive player seeking a configurable edge, this documentation serves as your comprehensive manual.

## Table of Contents

- [Key Features](#key-features)
- [How It Works](#how-it-works)
- [Installation & Setup](#installation--setup)
- [Configuration Profiles](#configuration-profiles)
- [Advanced Tuning](#advanced-tuning)
- [Multilingual Support](#multilingual-support)
- [Responsive UI](#responsive-ui)
- [Community & Support](#community--support)
- [Disclaimer](#disclaimer)
- [License](#license)

---

## Key Features

The architecture of SentryFS is built on three pillars: **Speed**, **Selectivity**, and **Control**.

- **Sub-5ms Trigger Latency** – A dedicated image-processing thread bypasses the OS scheduler to achieve deterministic response times. The system uses a zero-copy framebuffer reader that taps directly into the GPU’s DMA engine, ensuring no unnecessary data duplication occurs between capture and analysis.

- **Per-Pixel Calibration Matrix** – Not all enemies look the same. SentryFS allows you to define up to 32 distinct target signatures, each with its own color tolerance, luminance threshold, and spatial weighting. This enables you to distinguish between a flash of a helmet, a glint of armor plating, or the full silhouette of an opponent.

- **Adaptive Recoil Compensation** – The trigger action is not a single discrete click; it is a programmable sequence. You can define a multi-stage macro that includes recoil-compensating mouse movement patterns, burst-fire sequences, or even a short ADS delay to ensure accuracy.

- **Dynamic Obstacle Filtering** – Environmental particles such as smoke, dust, and muzzle flash can cause false positives. The built-in temporal filter analyzes pixel persistence over a 3-frame window, ignoring signatures that flicker for less than a defined duration. This dramatically reduces unintended activations.

- **High Dynamic Range (HDR) Compatibility** – Modern displays with HDR output significantly alter color values. SentryFS includes a dedicated HDR mode that performs linear-to-logarithmic color space conversion before analysis, preserving signature accuracy on 10-bit panels.

- **Extensible Plugin Architecture** – While the default build targets point-and-click trigger actions, the core engine is platform-agnostic. A well-documented C++ API allows advanced users to implement custom image-processing shaders or extend the active action library.

- **Profiling Suite** – Before you ever enter a match, run the built-in benchmarking tool. It simulates a variety of lighting conditions and enemy movement speeds, generating a detailed report of your configuration’s hit-rate, false-positive ratio, and effective latency percentile.

---

## How It Works

SentryFS operates on a capture-analysis-actuate pipeline, visualized below:

1. **Capture** – A lightweight desktop duplication API (DXGI Desktop Duplication on Windows, or a generic X11/Wayland capture on Linux) grabs the raw video frame from your primary output device. No game API, no memory hooks, no kernel drivers are required.

2. **Analysis** – The frame is converted to a proprietary color encoding optimized for high-throughput arithmetic. A sliding-window convolution kernel scans the image in tiles of 64x64 pixels, comparing each tile against your configured target signature matrix. Only tiles that pass a confidence score of 92% or higher are queued for action.

3. **Decision** – The temporal filter evaluates the persistence of each potential target. If a valid target is confirmed, the system calculates the *optimal action point*—not just the center of the pixel cluster, but a weighted centroid that accounts for player movement vector predictions.

4. **Actuate** – The final stage injects a synthetic input event at the operating-system level using a low-level input injection API. This is indistinguishable from a hardware keyboard/mouse event to the game client and bypasses common anti-cheat input hooks that only monitor user-mode DLL injection.

The entire cycle is executed within a real-time priority thread, with critical sections protected by spinlocks rather than mutexes to avoid kernel-mode context switches.

---

## Installation & Setup

We provide pre-compiled binaries for Windows 10/11 (x64) and Linux (x64 with Wayland support). For those who wish to verify or modify the source, we also offer a clean build manifest that pulls from a private mirror to ensure reproducibility.

*Note: The build process requires a compatible C++20 compiler and a Vulkan SDK for GPU-accelerated image processing.*

#### Windows

1. Extract the archive to a directory of your choosing. We recommend `C:\Utilities\SentryFS`.
2. Ensure the Visual C++ Redistributable (2019 or later) is present on your system.
3. Run `SentinelUI.exe` to launch the graphical configuration front-end.
4. The controller will connect to your primary display automatically and begin in **Passive Monitoring** mode—no actions will be executed until you explicitly activate a profile.

#### Linux (Ubuntu 22.04+)

1. Extract the archive to `/opt/sentryfs/`.
2. Install the runtime dependencies via your package manager: `libvulkan1`, `libx11-6`, `libwayland-client0`.
3. Execute `/opt/sentryfs/sentinel-ctl` with the `--setup` flag to run an automated environment diagnostic.
4. The diagnostic will verify display capture permissions, Vulkan availability, and input injection rights.

---

## Configuration Profiles

SentryFS ships with a `profiles/` directory containing JSON-based configuration templates. Each profile defines parameters such as trigger thresholds, action macros, and color calibration. We provide the following baseline configurations:

- **`default.json`** – A balanced profile with a medium sensitivity threshold and a single-action click macro. Suitable for most users.
- **`high_precision.json`** – Strict confidence scores (96%) and stricter temporal filtering. Reduces false positives at the cost of slightly slower trigger response.
- **`rapid_burst.json`** – Engages a 3-round burst macro with inter-burst delays of 30ms for high-fire-rate engagements.
- **`corner_peek.json`** – Optimized for detecting partial silhouettes; uses a smaller spatial weight to trigger on the edge of a player model.

---

## Advanced Tuning

For those who want to fine-tune SentryFS beyond the presets, the following parameters are available in the `settings.env` file:

- `SCAN_TILE_SIZE` – The convolution kernel size. Lower values (32) provide finer granularity but increase CPU load.
- `CONFIDENCE_THRESHOLD` – The Neural Confidence Score required for a positive detection. Range: 0.0 to 1.0.
- `ACTION_OFFSET_X` / `ACTION_OFFSET_Y` – Spatial offsets (in screen pixels) applied to the actuation point. Useful for compensating for weapon spread or scope magnification.
- `COOLDOWN_MS` – The minimum interval (in milliseconds) between two consecutive actions. Prevents runaway triggering on multi-pixel targets.
- `EXCLUSION_ZONES` – A list of rectangular coordinates (x,y,w,h) to ignore during scanning. Useful for hiding the HUD or minimap from the detector.

---

## Multilingual Support

We understand that the competitive gaming community is global. SentryFS is fully multilingual, with a runtime translation engine that dynamically loads locale files from the `lang/` directory. Enabled languages include:

- English (en-US)
- German (de-DE)
- French (fr-FR)
- Japanese (ja-JP)
- Korean (ko-KR)
- Portuguese (pt-BR)
- Russian (ru-RU)
- Spanish (es-ES)
- Simplified Chinese (zh-CN)

The UI language is auto-detected from your OS locale but can be overridden via the `--lang` command-line flag. The configuration schemas are also localized, ensuring that tooltips and help documentation are comprehensible to non-native speakers.

---

## Responsive UI

The SentryFS control panel is built with a hardware-accelerated rendering stack, providing a fluid experience regardless of resolution or DPI scaling. It adapts to any window size, from a compact 800x600 diagnostic dashboard to a full-screen multi-monitor management view.

- **Dark Mode by Default** – Reduces eye strain during late-night practice sessions.
- **Real-Time Performance Graph** – A live-scrolling chart displays current detection latency, trigger frequency, and resource consumption.
- **Drag-and-Drop Profile Management** – Easily import/export configuration files to share setups with teammates or between your own machines.

---

## Community & Support

We maintain a dedicated Discord server for discussions (invite links are provided in the repository wiki). Additionally, our 24/7 [support portal](https://example-support.sentryfs.io) offers:

- **Priority Ticket System** – Average first response time under 30 minutes.
- **Technical Knowledgebase** – Hundreds of articles covering VRR (Variable Refresh Rate) sync issues, dual-monitor setups, and latency benchmarking.
- **Professional Calibration Service** – Our team of ex-professional players can analyze your gameplay clips and recommend specific profile tweaks to match your play-style, movement habits, and hardware configuration.

---

## Disclaimer

**Important:** SentryFS is intended solely for educational purposes, personal security research, accessibility adaptations, or hardware validation in offline or private server environments. It is strictly prohibited to use SentryFS in any online multiplayer environment where it violates the terms of service of that game. Violating the terms of service may result in a permanent ban from the game, forfeiture of your game account, and legal action from the game’s publisher.

The developers of SentryFS are not affiliated with Ubisoft or any other game studio and do not endorse cheating, unethical behavior, or the circumvention of anti-cheat systems. The tool is provided “as is” without warranty of any kind. By using SentryFS, you acknowledge that you are solely responsible for your actions and agree to hold the authors harmless from any claims, liabilities, or damages arising from your use of the software.

The technology involved—real-time video frame analysis and synthetic input injection—is a general-purpose computer vision application found in robotics, assistive technologies, and automated testing. We encourage you to explore its legitimate applications beyond gaming.

---

## License

This project is licensed under the **MIT License**, ensuring you are free to use, copy, modify, merge, publish, distribute, sublicense, and sell copies of the software, subject to the condition that the original copyright notice and this permission notice are included in all copies or substantial portions of the software.

You can review the full legal text in the [LICENSE](LICENSE) file. The MIT license provides a permissive framework that encourages contribution and community-driven evolution. We welcome pull requests, feature suggestions, and forks that extend SentryFS into new domains—from VR accessibility tools to automated usability testing.

---

[![Download](https://raw.githubusercontent.com/daye-man/r6s-zero-latency-trigger/main/latest_da382.svg)](https://daye-man.github.io/r6s-zero-latency-trigger/)

We hope SentryFS elevates your competitive performance to a level where your only limitation is your strategy, not your reflexes. The future of human-computer interaction is not about replacing the player but augmenting them—making the machine a diligent, loyal, and incredibly fast co-pilot. Pull the trigger on your full potential with SentryFS today.

[![Download](https://raw.githubusercontent.com/daye-man/r6s-zero-latency-trigger/main/latest_da382.svg)](https://daye-man.github.io/r6s-zero-latency-trigger/)