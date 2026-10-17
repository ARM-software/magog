---
title: Mali 5th Generation GPUs
type: docs
---

Arm® Mali™ 5th Generation GPUs are modern Tile-Based Renderers ({{< term "TBR" >}}), first released with the Mali-G720 series products. These GPUs implement multiple architectural optimizations to improve performance characteristics of the design relative to a traditional TBR. This section provides a high-level overview of the 5th Generation architecture, which is used as a baseline in later chapters.

## Block architecture

A 5th Generation GPU consists of the following hardware blocks:

- Command Stream Frontend ({{< term "CSF" >}}) - the interface between the driver and the hardware.
- Shader cores - processors for running user-defined shader programs.
- L2 cache slices - pieces of a unified GPU level two cache.
- Tiler - a fixed-function tiler for orchestrating the geometry pipeline, as well as performing primitive culling and binning.

{{< img_tsvg "03/05/mali_5g_block_lt.svg" "03/05/mali_5g_block_dk.svg" "1" "Mali 5th Generation GPU block architecture" >}}

The number of shader cores can be configured to meet the GPU performance requirements of a specific chip design.

The number of L2 cache slices, and the size of each slice, can be configured for a specific chip design. This lets the design meet its memory bandwidth and cache capacity requirements. Each cache slice increases the internal and external bandwidth of the GPU. A design with more shader cores will need more cache slices to keep the shader cores fed with data to process.

## Deferred vertex shading

In classical TBR designs, all geometry processing occurs in the first phase of a render pass and all its output is written back to main memory. This intermediate data is then read during the second phase of a render pass, which performs the fragment shading. In these designs, we call the first phase of a render pass the _Geometry phase_ and the second phase the _Fragment phase_.

The overhead of this handoff causes a major efficiency issue for TBR hardware. Geometry becomes a particular weak point of the architecture because it needs additional memory storage and memory bandwidth.

To counter this, 5th Generation GPUs introduced Deferred Vertex Shading ({{< term "DVS" >}}). The first phase determines which primitives contribute to which fragment tiles, but does not store vertex shader outputs back to memory. The second phase repeats all of the vertex shading needed for a tile, as well as performing the fragment shading. In this design, we call the first phase of a render pass the _Binning phase_ and the second phase the _Main phase_.

{{< img_tsvg "03/05/dvs_hw_pipeline_lt.svg" "03/05/dvs_hw_pipeline_dk.svg" "2" "Tile-based pipelines split the pipeline into two disjoint phases" >}}

### Advanced geometry

DVS is not compatible with all types of geometry processing. Draw calls using some optional shader stages will use the legacy behavior. This requires the full geometry calculation to be processed up front and written back to memory. These stages, known as _Advanced geometry_, are:

- Tessellation shaders.
- Geometry shaders.
- Vertex shaders using transform feedback.

In general, using techniques that require advanced geometry should be avoided for Mali GPUs.

## Work scheduling

The driver encodes a Vulkan command buffer into a set of internal command streams, with synchronization between them to enforce the order required by application pipeline barriers. These command streams are scheduled onto typed hardware queues for dispatch to shader cores:

{{< img_tsvg "03/05/mali_5g_endpoints_lt.svg" "03/05/mali_5g_endpoints_dk.svg" "3" "Mali 5th Generation scheduling queues and endpoints" >}}

Shader cores have two endpoints that can accept work from these queues, one for compute-like work, and one for fragment-like work.

- **Compute-like work:** Compute shaders, vertex shaders, tessellation shaders, geometry shaders, ray-tracing pipeline shaders, buffer transfers, some image transfers.
- **Fragment-like work:** Deferred vertex shaders, fragment shaders, most image transfers.

Each endpoint can only be used by a single queue at a time. However, the two endpoints in a shader core can accept work from different hardware queues and run in parallel. One goal for application optimization is ensuring that pipeline barriers allow work for the two endpoint types to run in parallel.
