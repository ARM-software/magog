---
title: References
weight: 99
type: docs
sidebar:
  open: false
---

References are linked to directly from the text where needed, but here is a consolidated list of recommended reading, developer tools, and other useful sources of information.

## Documentation

The following developer tools and open-source projects are useful for developers working with Arm® GPUs:

* [Arm GPU Best Practices][DOC_GBP] - comprehensive Vulkan API best practice guidance for game engine developers.
* [Arm GPU Counter Documentation][DOC_LGC] - comprehensive performance counter documentation for all currently shipping Arm GPUs.

## Developer Tools

The following developer tools and open-source projects are useful for developers working with Arm GPUs.

### Proprietary tools

* [Arm Performance Studio][APS] - a suite of tools for profiling and debugging Arm GPUs in Android and Linux target devices.

### Open-source tools and libraries

* [libGPUInfo][LGI] - a library for querying Arm GPU properties.
* [libGPUCounters][LGC] - a library for sampling Arm GPU performance counters, including full documentation for third-party tooling integration.
* [libGPULayers][LGL] - a library of Vulkan layers, and some tooling for building new ones.
* [astcenc][AE] - a widely used compressor for offline {{< term "ASTC" >}} texture compression.

## Recommended reading

* [Real-time Rendering][RTR_BOOK], by Tomas Akenine-Möller, Eric Haines, Naty Hoffman, Angelo Pesce, Michał Iwanicki, and Sébastien Hillair.
* [Physically Based Rendering: From Theory to Implementation][PBR_BOOK], by Matt Pharr, Wenzel Jakob, and Greg Humphreys.
* [Real-time Shadows][RTS_BOOK], by Elmar Eisemann, Michael Schwarz, Ulf Assarsson, and Michael Wimmer.

[DOC_GBP]: https://developer.arm.com/documentation/101897/latest/
[DOC_LGC]: https://arm-software.github.io/libGPUCounters/

[LGI]: https://github.com/ARM-software/libGPULayers
[LGC]: https://github.com/ARM-software/libGPUCounters
[LGL]: https://github.com/ARM-software/libGPULayers
[AE]: https://github.com/ARM-software/astc-encoder
[APS]: https://developer.arm.com/Tools%20and%20Software/Arm%20Performance%20Studio

[RTR_BOOK]: https://www.realtimerendering.com
[PBR_BOOK]:https://www.pbrt.org/
[RTS_BOOK]: https://www.realtimeshadows.com/
