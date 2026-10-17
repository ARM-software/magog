---
title: Tile-based GPUs
weight: 3
type: docs
---

Mobile devices spend most of their time rendering simple user interfaces and lightweight games, which are proportionally light on geometry and heavy on fragment shading. Early mobile devices also used low-resolution screens, making them ideal targets for Multi-Sample Anti-Aliasing ({{< term "MSAA" >}}). The workloads that mobile GPUs needed to be most efficient at are the natural weaknesses of a classical Immediate Mode Renderer ({{< term "IMR" >}}).

Most mobile GPUs, including all Arm® GPUs, therefore implement a Tile-Based Renderer ({{< term "TBR" >}}). A TBR is designed to increase the energy efficiency of fragment shading by processing framebuffers as a stream of screen-space tiles. These tiles are small enough that the framebuffer working set can be kept in on-chip memory, close to the GPU.

To implement this approach, the GPU needs to know which primitives contribute to which tiles. TBRs implement a binning pass that assigns the primitives in a render pass to tiles before the fragment shading pass can begin. TBR hardware therefore splits the rasterization pipeline into two disjoint phases. The first phase consumes all vertices in a render pass and assigns primitives to tiles, a process commonly called binning. The second phase consumes the primitive binning data, performing fragment shading tile-by-tile, writing the final content of each tile back to memory.

{{< img_tsvg "03/03/tbr_hw_pipeline_lt.svg" "03/03/tbr_hw_pipeline_dk.svg" "1" "Tile-based pipelines split the pipeline into two disjoint phases" >}}

The disjoint stages of the hardware pipeline mean that workload scheduling for a TBR is more complex than an IMR. Applications can still achieve parallel processing with vertex shading and fragment shading running concurrently. However, this is now achieved by overlapping multiple render passes, which places more restrictions on scheduling than an IMR requires.

{{< img_tsvg "03/03/tbr_scheduling_lt.svg" "03/03/tbr_scheduling_dk.svg" "2" "Tile-based scheduling can pipeline processing across multiple render passes" >}}

## TBR advantages

The main strength of a TBR is that the working framebuffer for a tile is guaranteed to stay on chip. Any action touching the framebuffer during a render pass, such as depth and stencil testing or blending, will not consume memory bandwidth. Any transient values that only need to exist for the duration of the render pass can be discarded at the end. They do not need to be written back to memory at all.

The guarantee even extends to MSAA, because the on-chip tile memory is large enough to store multiple samples per pixel. Multi-sampled framebuffers can be resolved to a single sample when writing a complete tile back to memory, meaning that MSAA requires no additional framebuffer storage or memory bandwidth.

## TBR disadvantages

The main disadvantage of a classical TBR is that geometry handling is less efficient. On older Arm Mali™ GPUs, all vertex shader outputs must be written back to memory at the end of the first pipeline phase. They must then be read back in again during the second phase, causing high memory storage and bandwidth overhead.

Concurrent processing of vertex and fragment work to maximize hardware throughput is possible in a TBR, but it is more difficult to achieve than in an IMR. Application pipeline barriers must allow multiple render passes to run in parallel, and overly conservative barriers can cause serialization between the two phases.

{{< img_tsvg "03/03/bad_tbr_scheduling_lt.svg" "03/03/bad_tbr_scheduling_dk.svg" "3" "Conservative barriers can serialize phases of tile-based rendering" >}}

## Modern TBRs

Modern TBRs are more complex than the classical pipeline explained here. They use intelligent geometry processing to reduce how much data is written to memory and read back later. This avoids the worst of the geometry bandwidth problems.

This means that a modern TBR has some performance characteristics that resemble a classical IMR. The pipeline of the Arm 5th generation architecture GPUs is [explored later in this chapter][1].

[1]: {{< relref "./05_mali_fifth_gen_gpus.md" >}}
