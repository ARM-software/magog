---
title: What is performance?
type: docs
weight: 2
---

Before we dive into the technology, let us first review what we mean by performance, the reasons for poor performance, and how it can manifest.

## What do we mean by performance?

There are several performance metrics that you can use to measure how efficiently an application uses the GPU. It is usually beneficial to use multiple metrics to measure different behavioral aspects.

### Frame time

The most common metric that application and engine tooling uses is the time taken to render a frame, or some sub-element within it. This could be based on measuring time on the CPU, or using the graphics API to record timestamps as work runs on the GPU. Time is the most intuitive metric because it reflects what end users experience when running your application. However, it can also be tricky to use for mobile application development.

Production mobile devices use Dynamic Voltage and Frequency Scaling ({{< term "DVFS" >}}) to adjust clock frequencies based on the processing demands of the running workload. The technique aims to lower the frequency whenever possible to improve energy efficiency. Your optimizations may successfully reduce the size of a workload, but have no observable impact on frame time because the system reduces the frequency to match.

When using pre-production engineering devices, it is often possible to lock the clock frequencies to remove this timing variability. If you have access to a device that allows this, locking the CPU, GPU, and memory clock frequencies when profiling is certainly helpful. Unfortunately, few consumer devices expose this capability, even if the device has been put into developer mode.

> [!NOTE] No FPS please!
> End users often use frames per second (FPS) as a measure of performance, but it is a poor metric to use for technical work. FPS is not a linear metric, so statements like _"it improved by 1 FPS"_ are ambiguous and depend on where you started from.

### GPU cycles

To avoid the issues caused by frequency variability, another common approach is to measure the number of GPU cycles that were spent rendering a frame. This gives a direct measure of GPU cost that is less sensitive to clock frequency changes. This is the main metric that we recommend you use when measuring your application's rendering performance.

Without the ability to lock frequencies, there will still be some variability in measured results due to the second-order effects of any frequency changes.  For example, frequency changes can affect memory latency and the amount of memory bandwidth available per clock. Using frame GPU cycles is significantly better than trying to use frame time, but it is still not a perfect measure.

### GPU resource use

When optimizing for performance, you need to reduce your application's use of resources that are on the critical path. For example, if your shaders are saturating the texture unit, then optimizations will need to reduce the amount of texturing performed. Measure individual hardware resources, such as arithmetic operations, texturing operations, or memory accesses. These measurements show which parts of the workload should be optimized first.

When optimizing for power efficiency, whether to reduce thermal load or to improve battery life, it can be beneficial to reduce the use of any hardware resource. This often improves something that is not on the critical path. The overall GPU cycle use may not reduce, even if the optimization is beneficial. Using individual hardware unit metrics is the best way to validate that your optimizations actually simplified the workload.

### Memory bandwidth

GPUs are data-plane processors, and will inevitably read and write a considerable amount of data to and from system memory. Reducing the amount of memory accessed by a frame can help improve performance, and is particularly impactful for reducing power because accessing DRAM is energy intensive.

Measuring memory bandwidth consumed over a frame, measured as bytes read or written by the GPU per frame, is a great metric for monitoring frame complexity.

Measuring the instantaneous memory bandwidth, measured as gigabytes accessed per second, is useful when comparing the workload demand against the specified performance rating of the DRAM.

{{< callout type="info" >}}
High-end mobile chip designs use a System-Level Cache ({{< term "SLC" >}}) that sits between the GPU and the off-chip DRAM. GPU performance counters can only measure the amount of bandwidth between the GPU and the SLC, but have no visibility of the bandwidth between SLC and DRAM.
{{< /callout >}}

### Power consumption or device temperature

Mobile devices are battery powered and passively cooled, so another common optimization goal for more complex applications is to improve energy efficiency. This will give your users longer battery life, and make it less likely that their device will overheat and throttle performance.

Developers commonly request the ability to measure power consumption and device temperature during profiling runs. Application APIs available in Android can only present a coarse thermal state and battery percentage. They are indicative hints but, in my experience, are not precise enough for detailed technical work.

Android devices can optionally support [on-device power measurement][1], which can give battery and per-rail power measurements, but availability depends on the device OEM integrating support for it.

{{< callout type="info" >}}
The Google Pixel devices have supported reporting per-rail power metrics since the Pixel 6. We'll be using the Pixel 7 as one source of performance and power efficiency data for the case studies. This device has an Arm® Mali™-G710 GPU.
{{< /callout >}}

## What limits performance?

There are multiple different classes of problem that can cause an application to run slowly, and they each have very different solutions.

### Workload too large

The first class of problem is one of scale. The workload may be perfectly efficient, allowing the hardware to run at full throughput, but too large to reach the desired performance. This is a common problem that developers encounter when trying to broaden the reach of an existing application to a wider range of devices.

The solution to this class of problem is to reduce the size of the workload. This can be achieved by:

* Reducing the number of workload elements that need processing, for example by reducing model vertex count or render pass resolution.
* Reducing the complexity of individual workload elements, for example by optimizing the existing implementation or changing to a smarter algorithm.

The skill in doing this well is applying a size reduction while preserving the visual quality of the final render.

### GPU too idle

The second class of problem is one of scheduling. The workloads that make up a frame may be individually perfectly efficient, but some form of scheduling restriction means that they are not running in parallel. Available hardware is being left idle, and potential performance is unused.

> [!NOTE]
> Serialized scheduling can apply between CPU threads, between CPU and GPU, and between GPU hardware queues.

The solution to this type of problem is to find the cause of the serialization, and then refactor to avoid it. There are four common types of change that can be needed to improve GPU serialization:

* Moving slow CPU workloads off the critical path, so that they stop blocking the main application render from submitting work to the GPU.
* Moving workloads that wait on the outcome of an earlier workload later in the issue schedule, so that synchronization is less likely to block and wait.
* Relaxing pipeline barrier constraints, so that the command stream allows GPU hardware work queues to progress independent workloads in parallel.
* Removing CPU-side waits on GPU tasks that cause the GPU hardware queues to drain, so that the queues do not go idle.

### Workload too inefficient

The final class of problem is one of execution inefficiency. The workload has been scheduled on the hardware, but it is not making the best use of the available resources. Inefficiencies could be causing additional processing or memory bandwidth, or could be causing idle bubbles (short periods of underutilization) due to resource contention inside the hardware.

The solution to this type of problem is to identify the specific cause of the inefficiency, and then refactor to avoid it. There are too many possible inefficiencies to list here, but we will cover them extensively in the later chapters of this book.

## Why performance is lost

There are really two fundamental causes of the problems described in the previous section.

### Design losses

Mobile devices span a wide range of performance points. It takes a bit of ahead-of-time planning to create an application that smoothly scales its complexity to reach its target performance across devices with different capabilities.

The first way that a project can have performance problems is an initial design that is too complex for some of the target devices. In this scenario, no implementation could meet the design goals on those devices while also staying within the available performance budget.

Fixing a poor design late in the project often requires some rather blunt fixes, such as reducing asset complexity, reducing resolution, or disabling rendering effects. These can recover lost performance, but often with reduced visual fidelity and needing additional development time to rework the content.

No design is perfect, and some minor rework is inevitable, but a little early planning around device support can help avoid major problems later.

### Implementation losses

The second way that a project can have performance problems is inefficient implementation of a design. The design could reach the performance target, but implementation losses mean that performance has dropped below target.

If you are lucky, the performance loss will be caused by a small number of issues that individually have a major performance impact. One or two fixes of this nature can recover a lot of performance very quickly!

However, complex application frames require thousands of API calls, art assets, and GPU programs to render the final output image. The sheer number of unique elements means that performance is more often lost to "death by a thousand cuts". Many small implementation inefficiencies can add up to a significant overall performance loss. It doesn't take that many 1% inefficiencies to turn a 60 FPS game into a 30 FPS one.

{{< img_tsvg "01/02/percent_losses_lt.svg" "01/02/percent_losses_dk.svg" "1" "Cumulative impact of seventy 1% inefficiencies" >}}

## Getting performance back

New projects can often have one or two large performance issues that stand out in profiles, and fixing those can give significant improvements. However, those are the types of problem that get fixed early in the project and tend not to recur after they have been fixed.

For projects that are past that phase, we would love to say that there is a quick fix. If there is, we've not found it yet. In our experience, the applications with the best performance and visual quality have been carefully honed. Each piece visually justifies its cost, and the pieces are optimized and fit together cleanly.

If the problem that commonly causes poor performance is "death by a thousand cuts", the corresponding fix should be no surprise. It is "polish by a thousand improvements". The good news is that this actually makes the problem a lot more manageable!

If we asked a developer if they could double the performance of their application, they would probably say it was impossible. If we asked the same developer if they could improve performance by 1%, they could probably give us a handful of ideas to try without much thought.

Design changes and configuration changes can certainly give big improvements, and are important to get right early in your project. Once things have stabilized, you must polish out the small inefficiencies, and then repeat until you achieve your performance target.

### My experience with astcenc

I took over as maintainer of the Arm® offline {{< term "ASTC" >}} texture compressor, ASTC Encoder (`astcenc`), at the end of 2019. At the time it was slow. Multiple developers complained that it was more than an order of magnitude behind the compressors used for the BC and ETC format families.

A week or two after taking over the project, one of the directors in the Arm ecosystem team asked me how much faster we could make it. My answer was _"... perhaps two or three times faster."_ I honestly had no idea how we were going to do it. It still wasn't enough to be competitive, but it seemed like a reasonable goal and I could see a route to getting started.

In August 2020, we released astcenc 2.0 and it was between two and three times faster than the 1.7 release we started with. Mission accomplished! Even though the overall improvement was large, the individual optimizations that got us there were all small. The vast majority were 1-2% improvements, and I can count those that were larger than 5% on the fingers of one hand. The hero of this story is incremental improvement, chipping away at a big problem one small optimization at a time.

I found doing this style of optimization was a great way to learn the codebase, and discover what helped and what didn't. When implementing one idea, I would spot opportunities to apply it elsewhere. Sometimes I would find a new way to improve the algorithm or implementation that I was refactoring. The incremental optimizations became self-fueling — more ideas, giving more improvements, giving more ideas.

By the end of 2024, the compressor was 30 to 40 _times_ faster than the original compressor. Even at that scale of increase, most improvements we made were still small and incremental. Big refactors that give big improvements have a place, especially if they allow a pivot to a better algorithm. However, don't undervalue the impact that a steady stream of small improvements can have.

[1]: https://github.com/google/perfetto/blob/main/docs/data-sources/battery-counters.md
