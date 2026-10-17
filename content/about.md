---
title: About MAGOG
type: about
---

The Mobile Application GPU Optimization Guide is an eBook that helps you optimize your mobile application. It focuses on getting the best performance out of the GPU in your device.

## What does MAGOG mean?

The acronym was inspired by the Gog Magog hills, or "the Gogs" to anyone living locally. These are low rolling chalk hills to the south of Cambridge in the UK. The north edge of the Gogs is visible from Pete's office in the Arm Cambridge campus.

{{< img_rast "about/thegogs.webp" "1" "The Gogs, as seen from the Arm Cambridge office" >}}

## Technical details

### License

The prose and images of this book are published under the [Creative Commons Attribution 4.0 International license](https://spdx.org/licenses/CC-BY-4.0.html) (SPDX: CC-BY-4.0).

Source code snippets inline in the documentation, in any standalone examples, and for the web page framework, are licensed under the [MIT license](https://spdx.org/licenses/MIT.html) (SPDX: MIT).

### Reporting errors

If you find any typos or errors, please raise an issue on the [GitHub issue][4] tracker. The document is regularly updated and we prioritize bug fixes.

### Contributing content

If you would like to make a contribution, please get in touch using the GitHub issue tracker before starting. Contributions are welcome, but we want to ensure that they will not collide with topics that are already under development, and fit in with our content roadmap.

All contributions are made under the license used for this project.

## Contributors

We would like to thank the following contributors for their help authoring and editing this book and its supporting code samples.

### Pete Harris, Arm

Pete is a Distinguished Engineer in the Arm GPU architecture team.

He has worked in mobile development since before phones were smart, and has worked on Arm® Mali™ GPUs since 2009. He has led GPU performance methodology and software optimization teams, created multiple GPU profiling tools and libraries, and helped many game studios successfully optimize their games.

He is the maintainer for multiple Arm-sponsored open-source projects related to mobile graphics and GPUs, including this book. Other projects include:

* [astcenc][3] - a widely used compressor for offline {{< term "ASTC" >}} texture compression.
* [libGPUCounters][1] - a library for Arm GPU performance counter sampling.
* [libGPULayers][2] - a toolkit for creating Vulkan layers, and some tooling layers built with the toolkit.

### Panagiotis Christopoulos Charitos, Arm

Panagiotis is a Principal Software Engineer in the Arm graphics platforms team.

His current focus is on researching modern real-time graphics techniques, including GPU-driven rendering, ray tracing, mesh shaders, and work graphs. His work spans graphics APIs, graphics drivers, rendering techniques, and engine optimization.

He has previously worked on Arm Mali driver development for OpenGL ES and Vulkan. He has also collaborated with game engine vendors to analyze performance issues and optimize their engine for Mali GPUs.

### José Emilio Muñoz López, Arm

José Emilio is a Staff Software Engineer in the Arm graphics platforms team.

He has worked on Arm Mali GPUs since 2015, contributing to driver instrumentation, tooling, and assisting ecosystem partners to optimize for Mali. He currently works on modern graphics APIs and advanced rendering techniques for mobile GPUs, including Vulkan, ray tracing, and neural graphics.

[1]: https://github.com/ARM-software/libGPUCounters
[2]: https://github.com/ARM-software/libGPULayers
[3]: https://github.com/ARM-software/astc-encoder
[4]: https://github.com/ARM-software/magog/issues
