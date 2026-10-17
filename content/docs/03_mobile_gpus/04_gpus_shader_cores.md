---
title: GPU shader cores
weight: 4
type: docs
---

The rasterization pipeline defines the macro-scale processing approach for graphics, combining user-defined shader stages and fixed-function stages. Shader stages are software-defined, running program code provided by the application on a shader core. A modern GPU shader core is a processor designed for data-parallel processing, and makes different design choices compared to a typical CPU.

In this section we compare and contrast some of the architectural concepts that underpin contemporary high-performance CPU and GPU design.

## CPU essentials

CPU instruction sets generally define a serial instruction stream, where each instruction behaves as if the previous instruction has already completed.

CPUs can support coarse parallelization by splitting work over multiple threads, but many workloads are limited in how many threads they can use. Building a processor that can execute these workloads faster means that a single thread's instruction sequence needs to run faster. Let us look at how they do that.

### Processor essentials

As a starting point, let us consider a very basic processor that can simply handle data processing operations.

{{< img_tsvg "03/04/core_basic_lt.svg" "03/04/core_basic_dk.svg" "1" "A simple three stage pipeline" >}}

This executes an instruction in three pipeline stages:

1. Read inputs from the register file.
2. Perform the required calculation.
3. Write the result back to the register file.

This is perfectly functional and resembles the kind of CPU architecture you would find in a microcontroller. However, it does not qualify as high-performance, so we need to add hardware to the design to make it faster.

### Increasing clock frequency

The first change we can make to increase performance is to increase the clock frequency. If we can clock the processor faster, we can directly increase instruction throughput.

Clock frequency is limited by the amount of logic in each pipeline stage, so CPU designs pipeline instruction execution over multiple stages. This reduces the logic depth in each pipeline stage, increasing the maximum frequency that can be achieved.

{{< img_tsvg "03/04/core_pipelined_lt.svg" "03/04/core_pipelined_dk.svg" "2" "A pipelined five stage pipeline" >}}

### Adding forwarding paths

The downside of pipelining instruction execution is that there are now multiple cycles between issuing an instruction and getting its result back. To benefit from the higher throughput, this design needs to issue instructions while earlier instructions are still running. If an instruction needs the result of an earlier instruction, it might need to stall the pipeline. The stall lasts until the earlier instruction completes.

{{< img_tsvg "03/04/core_bubble_lt.svg" "03/04/core_bubble_dk.svg" "3" "A two cycle idle bubble due to instruction result dependencies" >}}

Data dependency stalls reduce the performance of the system, so high-performance designs use forwarding paths within the pipeline to pass results to following instructions. This reduces the number of stalls, at the cost of added hardware data paths and control logic to manage the forwarding.

{{< img_tsvg "03/04/core_pipelined_fwd_lt.svg" "03/04/core_pipelined_fwd_dk.svg" "4" "A pipeline with added internal forwarding paths" >}}

### Adding branch prediction

Software contains control flow instructions, which change which instructions in the instruction stream are executed. Most branches are conditional, meaning that the processor needs to know the result of an earlier instruction to determine whether a branch is taken. This introduces another kind of dependency stall, which would reduce performance if we always just blocked and waited for the branch condition to be available.

Processors add branch prediction hardware to make an educated guess about whether a branch will be taken, and keep issuing instructions based on this guess. If the guess was correct, the processor stays busy and gains performance. If the guess was wrong, the incorrectly issued instructions need to be canceled, and the pipeline refilled with the instructions from the correct control path.

{{< img_tsvg "03/04/core_pipelined_bp_lt.svg" "03/04/core_pipelined_bp_dk.svg" "5" "A pipeline with added branch prediction" >}}

### Adding data access prediction

If a memory load misses in the L1 data cache, it can take many cycles to get a result from a more distant cache or DRAM. Cache misses nearly always mean that the processor will stall because there simply isn't enough non-dependent work to issue while the CPU is waiting.

To avoid this, CPUs often use data prefetchers to guess what data the program is likely to need next. These will preload the data into the cache ahead of time, so that it is more likely to be immediately available when the processor needs it. This improves performance but adds more control logic to the CPU. It also increases memory bandwidth because some guesses are wrong and fetch data that is not actually needed.

{{< img_tsvg "03/04/core_pipelined_pf_lt.svg" "03/04/core_pipelined_pf_dk.svg" "6" "A pipeline with added data prefetch" >}}

### Adding superscalar execution

Most instruction sets define a serial instruction stream, but there will be cases where neighboring instructions are not dependent on each other. We can exploit this to issue multiple instructions in parallel in a single clock cycle. However, this needs additional control hardware to detect and issue the non-dependent instructions.

{{< img_tsvg "03/04/core_pipelined_ss_lt.svg" "03/04/core_pipelined_ss_dk.svg" "7" "A pipeline with added superscalar issue" >}}

### Adding SIMD vector execution

Data processing algorithms often want to operate on vectors of data, performing an identical operation on multiple data items. CPUs frequently support this using Single Instruction Multiple Data ({{< term "SIMD" >}}) vector instructions, such as the Arm NEON or x86-64 SSE instruction sets. SIMD vectorization is more efficient than performing the equivalent scalar operations because the control overhead of decoding and issuing instructions is amortized over multiple data processing operations.

Unfortunately SIMD vectorization is also highly situational. Some software may not be able to use SIMD vectorization at all, and some may only be able to partially utilize the available vector width.

{{< img_tsvg "03/04/core_pipelined_simd_lt.svg" "03/04/core_pipelined_simd_dk.svg" "8" "A pipeline with added vector data paths" >}}

### Adding out-of-order execution

Static superscalar instruction issue exploits localized parallelism in the instruction sequence, but is limited to issuing instructions in program order. As soon as a dependency is encountered in that small instruction window, it falls back to scalar issue. This limits how much parallelism it is able to find and exploit.

High-performance processors use out-of-order execution to find independent instructions in a much larger instruction scheduling window. In a modern processor, this window is commonly hundreds of instructions long. This significantly increases the amount of parallelism that can be found to keep the CPU pipelines busy. However, it also significantly increases the amount of control logic needed to manage the scheme.

{{< img_tsvg "03/04/core_pipelined_ooe_lt.svg" "03/04/core_pipelined_ooe_dk.svg" "9" "A pipeline with added out-of-order instruction issue" >}}

### Adding a big cache

Accessing data from external DRAM is a slow operation. The DRAM fetch latency observed by a mobile CPU can be more than 125 ns, or more than 375 CPU cycles at 3 GHz. Even a small number of cache misses that require a DRAM fetch can significantly impact performance. For a CPU with 4-wide instruction issue, 375 cycles is 1500 issue slots. No real-world software has that many non-dependent instructions to issue, so cache misses that need to fetch from DRAM nearly always cause a significant processor stall.

CPUs use large caches for the outer level of the CPU cache hierarchy, relative to the size of their total working set. They also benefit from a large System-Level Cache ({{< term "SLC" >}}). The aim here is to keep the majority of the working set close to the CPU, avoiding as many DRAM fetches as possible.

{{< img_tsvg "03/04/core_pipelined_cache_lt.svg" "03/04/core_pipelined_cache_dk.svg" "10" "A pipeline with added large caches" >}}

### Putting it all together

The last diagram should make it clear that high-performance CPUs are possible. However, they need extra hardware that isn't doing useful data processing.

This extra hardware has a cost, both in terms of increased silicon area and reduced runtime energy efficiency. We can make CPUs that are fast, or we can make CPUs that are energy efficient, but a single design cannot have both at the same time. This is the reason for the heterogeneous mix of CPUs found in modern systems. The operating system can choose between high-performance or high-efficiency cores for different workloads based on their performance demands.

## GPU shader core essentials

GPUs are accelerators for data-parallel problems that are structured very differently from traditional CPU software. The graphics and general-purpose GPU ({{< term "GPGPU" >}}) problem domains are commonly called "embarrassingly parallel"; even simple games have hundreds of thousands of vertices and millions of fragments to process. This level of abundant parallelism enables a fundamentally different approach to building the programmable shader core.

### Adding a barrel pipeline

In our archetypal CPU we only have one thread to run, so filling the pipeline requires issuing sequential instructions from a single thread. This causes many problems with result dependencies between instructions that require additional hardware to manage. Shader cores still want to use data path pipelining to increase achievable clock frequency, but choose a different approach to select which instructions to issue.

Consider a hypothetical core design with a 3-stage data path and that can also support 3 concurrently running threads. Instead of trying to find parallelism within a single thread's instruction stream, the core simply selects a different thread to issue each cycle. This architecture is known as a barrel pipeline, where the scheduler cycles through runnable threads in a fixed rotation.

{{< img_tsvg "03/04/core_barrel_lt.svg" "03/04/core_barrel_dk.svg" "11" "A three stage barrel pipeline" >}}

This one change avoids most of the instruction dependency issues that a high-performance CPU encounters, removing the need for many of the hardware blocks. You can see the effect in the pipeline diagram in Figure 11. Each thread is guaranteed to only have a single instruction in flight at any point in time.

There is no need for data forwarding within the pipeline. By the time instruction N+1 for a thread is issued, instruction N has already completed and produced its result.

There is no need for branch prediction. By the time we need to pick instruction N+1, instruction N has already completed and produced its condition flags.

There is no need for superscalar issue or out-of-order execution. We do not need to find parallelism, the GPU has it already in the thread pool. To support issuing to two parallel pipelines, we can just double the number of threads in the core. The core can then pick two different threads each issue cycle.

This approach also makes GPUs much less sensitive to instruction latency and instruction sequencing than a CPU. As a result, many optimizations that are commonly applicable to CPU software do not apply to shader programs.

{{< callout type="info" >}}
The first use of a barrel pipeline that we know of is in the Control Data Corporation 6600 mainframe. The CDC 6600 was designed by the legendary computer scientist Seymour Cray, and is widely considered to be the first supercomputer.

{{< img_rast "03/04/CDC-6600.webp" "12" "CDC 6600 mainframe (Jitze Couperus, CC BY 2.0)" >}}

The CDC 6600 used ten banks of ferrite core memory, which were accessed via a dedicated peripheral processor. The peripheral processor used a ten-cycle barrel pipeline to rotate through the memory banks.
{{< /callout >}}

### Adding massive multithreading

Having enough threads in the core to fill the data path solves the problems caused by short-lived instruction-level dependency stalls. However, because GPUs run a data-heavy workload, we expect high-latency data cache misses.

GPUs solve this problem by adding even more threads, creating a massively multithreaded architecture. A GPU core can support thousands of resident threads, most of which can be stalled waiting for data from memory. As long as we have enough active threads to issue to the processing pipelines, cache misses have little performance impact. This remains true even if a significant proportion of the thread pool is blocked.

{{< img_tsvg "03/04/core_barrel_tp_lt.svg" "03/04/core_barrel_tp_dk.svg" "13" "A barrel pipeline with a massive thread pool" >}}

This approach makes GPUs less sensitive to memory latency than a CPU. As a result, many CPU optimizations for instruction scheduling and data layout do not apply to shader programs.

### Adding SIMT vector execution

CPU vector hardware uses SIMD vectorization, which exposes vector instructions to each thread and requires the compiler to generate those instructions to get the benefit. When it is applicable to a program, it works well. However, SIMD vectorization depends on the program containing vectorizable work that can be packed into those instructions.

GPUs want the efficiency advantages of vector hardware, because it allows multiple data processing operations to share the control overhead of a single instruction. However, GPUs also want to avoid the situational applicability of SIMD vectorization. This is another area where we can exploit the data-level parallelism present in GPU workloads.

Within a single draw call or a compute dispatch, we have many threads and every thread runs the same shader program. We can fill the width of vector hardware by issuing the same scalar instruction for multiple threads that are running the same shader. This is known as Single Instruction Multiple Thread ({{< term "SIMT" >}}) vectorization. A group of threads running in lockstep is commonly called a warp, a wave, or a subgroup.

This style of vectorization lets the hardware benefit from the efficiency of vector hardware. It also stays widely applicable because the compiler is generating a scalar instruction stream.

#### Shared data fetch

Threads in a warp are issued together for arithmetic instruction execution but, outside of specific cross-lane operations, processed independently. However, threads in a warp are often spatially adjacent and share data, so this style of vectorization can also make memory access more efficient.

#### Branch divergence

A new downside introduced by SIMT vectorization is a loss of efficiency if threads in a warp have divergent control flow. The threads in a warp run in lockstep, issuing the same instruction for all threads. If threads within a warp follow different control paths, the GPU will need to mask data lanes on the wrong path. This causes an efficiency loss because it is an effective reduction in the active vector width.

{{< img_tsvg "03/04/simt_divergence_lt.svg" "03/04/simt_divergence_dk.svg" "14" "Divergence in a 4-wide SIMT warp, with three threads taking the if and one thread taking the else" >}}

## Designing out Amdahl's law

As should now be clear, GPU shader cores rely on a large number of concurrently running threads for good performance. This approach only works if threads do not need to synchronize with other threads.

Software developers are often aware of Amdahl's law. It describes how program performance is expected to scale with core count, based on the amount of parallelizable code it contains. If an application contains 20% serial code and 80% parallel code, doubling from 1 to 2 threads reduces runtime to 60% of the original. Doubling again to 4 threads only reduces runtime to 40% of the original, because the serial 20% proportion starts to dominate.

{{< img_tsvg "03/04/amdahls_law_lt.svg" "03/04/amdahls_law_dk.svg" "15" "Performance scaling based on Amdahl's law" >}}

GPUs want to run with thousands of threads. It should be clear that this means that they have almost no tolerance for serial code - any point of serialization would destroy their performance. GPU APIs are therefore designed to prevent, or at least limit the scope of, cross-thread data sharing and synchronization.

Most inputs are read-only, most outputs are write-only, and most shader threads are computationally independent of other threads in the same dispatch. Where data sharing or synchronization across threads is possible, it is deliberately constrained to ensure that performance impact is managed. For example, you can share data across threads within the small pool of threads in a compute work group, but no wider. These limitations are a deliberate attempt to prevent the serialization points that would kill performance.

Modern APIs are weakening the strength of restrictions here. However, applications that make heavy use of cross-thread synchronization and shared data access will reduce their performance. Amdahl's law will reassert itself and again limit scaling.
