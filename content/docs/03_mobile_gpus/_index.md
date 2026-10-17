---
title: Mobile GPUs
weight: 3
type: docs
sidebar:
  open: false
---

A mobile GPU must balance three competing constraints: rendering performance, energy efficiency, and silicon cost. It needs enough compute and memory bandwidth to render demanding workloads. However, it must do so within the tight power, thermal, and silicon area limits of a mobile device.

These constraints mean that most mobile GPUs use a Tile-Based Rendering architecture ({{< term "TBR" >}}), not the Immediate Mode Rendering architecture ({{< term "IMR" >}}) found in desktop PCs and game consoles. This choice brings different performance trade-offs that application developers must understand to make effective use of the GPU.

This chapter introduces essential GPU architecture concepts that we will refer back to in later technical topics. The goal of this chapter is not to explain everything in perfect detail, but to give a mental model of how everything fits together. It also explains what makes mobile GPUs different from the GPUs that you may have used in the past.

This chapter includes:

* [Rasterization pipeline][1]: The core stages of the graphics pipeline, and how they turn application data into pixels.
* [Immediate mode renderers][2]: How traditional desktop-style renderers process geometry and fragments, and the pros and cons of the approach.
* [Tile-based renderers][3]: How mobile renderers use tiling to reduce framebuffer bandwidth, and the trade-offs this creates for applications.
* [GPU shader cores][4]: Why programmable GPU shader cores are built for throughput rather than single-thread latency.
* [Arm® Mali™ 5th Generation GPUs][5]: A high-level look at the latest Arm GPU architecture, its major hardware blocks, and its programming model.

[1]: {{< relref "01_rasterization_pipeline.md" >}}
[2]: {{< relref "02_immediate_mode_renderers.md" >}}
[3]: {{< relref "03_tile_based_renderers.md" >}}
[4]: {{< relref "04_gpus_shader_cores.md" >}}
[5]: {{< relref "05_mali_fifth_gen_gpus.md" >}}
