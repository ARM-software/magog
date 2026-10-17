---
title: Data matters
weight: 3
type: docs
---

Processing on a CPU or GPU consumes two main types of resource, computation and data. Computation defines the processing that needs to occur inside the CPU or GPU. Data defines the memory store and memory bandwidth needed to supply the inputs and store the outputs of that processing.

CPU-based workloads are typically control-plane workloads; they involve complex stateful computation operating on relatively small amounts of data. Optimizing software for a CPU therefore tends to focus on improving code efficiency, and making sure the small data working set makes good use of caches.

GPUs are designed to accelerate a different class of problem. GPU workloads are data-plane workloads; they process large volumes of data with relatively little computation per byte of data. Optimizing for a GPU still requires efficient code efficiency in your application's shader programs, but ensuring efficient data encoding and data use is just as important.

## GPU memory hierarchy

A GPU, like a CPU, has a hierarchy of caches to keep data close to where it is needed. Data that is closer to the hardware that uses it can be accessed with lower latency and lower energy per access.

Unlike a CPU, a GPU's caches are small relative to the amount of data that it processes. High-end Arm® Mali™ GPUs are commonly configured with 2–4 MB of level 2 (L2) cache. A complex mobile game can require more than a hundred megabytes of data per frame. A GPU L2 is never going to be large enough to cache an entire frame. Instead, the L2 is sized to keep localized access hot spots resident in the cache. The GPU must continuously stream data in and out of the cache as this working set moves across the frame.

{{< img_tsvg "02/03/gpu_caches_lt.svg" "02/03/gpu_caches_dk.svg" "1"  "An example GPU memory hierarchy"  >}}

High-performance mobile systems increasingly include a System-Level Cache ({{< term "SLC" >}}), a large cache that sits between the DRAM and the processors in the {{< term "SoC" >}}.

Mobile SLC capacities continue to grow, with high-end devices now including more than 10 MB of cache. This is large enough to effectively cache GPU resources for short periods within a frame, but it cannot keep resources on-chip across multiple frames. GPU workloads inevitably require energy-intensive off-chip DRAM accesses.

## Logic power vs DRAM power

Most developers with some experience optimizing software for CPUs understand the performance implications of cache misses. Memory that is farther from the processor in the memory hierarchy has higher latency, and longer access times will naturally impact performance.

What is less widely known is that DRAM accesses are very energy-intensive, often being an order of magnitude more expensive than SLC accesses. Reducing DRAM bandwidth is an effective way to improve the performance of a thermally limited application, or to improve device battery life.

In addition, there is an energy efficiency gap between DRAM and logic circuits that is widening over time. Over the last decade we have seen more than a 10x efficiency improvement in logic CMOS silicon process, but less than a 2x improvement in LPDDR DRAM.

{{< img_tsvg "02/03/logic_vs_memory_lt.svg" "02/03/logic_vs_memory_dk.svg" "2" "Relative silicon process logic vs DRAM power efficiency improvements between 2012 and 2024" >}}

This efficiency gap comes from the physics of the silicon process alone, and another gap of similar magnitude comes from the GPU designs themselves. Over time, Arm GPUs have improved GPU processing efficiency faster than they have optimized workload memory bandwidth.

When all of these factors are put together, this means that some applications spend more power on DRAM access than on the GPU itself. Application data efficiency matters, and it matters more every year.

## Algorithm logic-to-bandwidth balance

The disparity in efficiency between logic and DRAM means that high-performance mobile devices are most efficient when processing arithmetic-heavy workloads. The mobile thermal budget balancing act means that mobile GPUs have an ample supply of energy-efficient {{< term "FLOP/s" >}}, but less memory bandwidth per arithmetic operation to feed them.

It is often said that the capabilities of a premium mobile phone are roughly equivalent to the previous generation of game consoles. This serves as a crude guideline, but the reality is more nuanced.

Performance figures[^1] for the Sony PlayStation®4 GPU state that it has 1.84 teraflops (TFLOP/s) of {{< term "FP32" >}} performance, and 176 GB/s of memory bandwidth. This gives a ratio of 10.5 {{< term "FLOPs" >}} per byte of memory access, which is indicative of the workload balance that PS4™ shaders are tuned for.

A mobile device using an Arm Immortalis™-G925 GPU, configured with 12 shader cores clocked at 1 GHz, has more than 3 TFLOP/s of FP32 performance. However, in a thermally constrained environment, the device only has a thermally sustainable bandwidth budget of approximately 30 GB/s. This gives a ratio of 102 FLOPs per byte of memory access. Attempting to run a workload tuned for 10.5 FLOPs per byte on this GPU will end up memory-bound and unable to use the arithmetic performance available.

{{< img_tsvg "02/03/mobile_vs_console_lt.svg" "02/03/mobile_vs_console_dk.svg" "3" "Mobile arithmetic and memory performance compared to previous console generations" >}}

Premium mobile devices can supply more than 60 GB/s of usable bandwidth for short bursts. However, they cannot sustain this bandwidth across a whole frame within the available thermal budget. This additional burst bandwidth is indicated by the hatched region in Figure 3.

[^1]: https://en.wikipedia.org/wiki/PlayStation_4_technical_specifications
