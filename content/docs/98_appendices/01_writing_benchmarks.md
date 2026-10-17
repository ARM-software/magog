---
title: Writing benchmarks
weight: 1
type: docs
---

Most profiling and optimization work will use your full application, because often issues only accumulate into a problem when everything is run together. However, when researching a technique or optimizing a known problem area, it is often more efficient to extract a synthetic microbenchmark to work on.

Microbenchmarks are usually intended to isolate a specific behavior, allowing you to develop and optimize it outside of the full application. The smaller project means that you can iterate faster, and data gathered by running it will be easier to understand.

While writing a benchmark is not difficult, it is also very easy to write a bad one that can give misleading results. We would like to share a few useful tips and tricks that can help you ensure that you are measuring a representative workload.

## Good microbenchmarks

A good microbenchmark is one that behaves and performs the same as your real project would, when taken in the context of the full application. Conclusions taken from a good microbenchmark are robust, and translate well into the full game.

A bad microbenchmark is one that does not behave or perform the same, meaning that profiling and optimizing it can give misleading results. Optimizations may not help when translated into the full game implementation.

It is surprisingly easy to accidentally make a bad microbenchmark, often because you oversimplify the workload which allows the hardware to use an easier execution path. The tips in this section can help you avoid these problems.

### Setting frame complexity

As noted in earlier chapters, mobile devices use {{< term "DVFS" >}} to adjust the GPU frequency to match the workload processing demand. Workloads can behave differently at different frequencies. Ideally, a frame of your microbenchmark should have a similar total cost to a full frame of your game, so the frequencies match.

Microbenchmarks are usually small slices of a renderer, and so run much faster than a full game frame. When increasing the cost of a microbenchmark to reach a particular GPU frequency, it is important that you do not accidentally alter what the microbenchmark is measuring.

> [!NOTE] A bad benchmark
> A useful performance metric for a GPU is its user interface fill rate, measured as the number of textured and blended fragments it can retire per clock cycle. One widely used user interface benchmark increased its cost by adding blended layers to a single render pass until the performance reduced to the target frame rate. Each layer was issued using its own draw call and loaded data from a unique texture.
>
> The end result of this dynamic layering was that a high-end GPU could be asked to render dozens of full screen layers. The benchmark workload was no longer representative of the UI workload it claimed to represent, and was no longer measuring fill rate. It had become a stress test for draw call and texture descriptor caching.

The most effective way of increasing the cost of a microbenchmark is just doing more of it, running multiple representative render passes or compute dispatches back-to-back.

### Setting test duration

It can take a few seconds for DVFS to stabilize on its chosen operating point. When designing microbenchmarks, run each test for at least 15 seconds and discard the first 5 seconds of data.

### Avoiding optimistic framebuffer compression

Arm® GPUs can use Arm FrameBuffer Compression ({{< term "AFBC" >}}), a lossless image compression format, to compress framebuffers written by a render pass. AFBC is a variable bitrate compression, and will use less bandwidth to store simpler images.

If your microbenchmark needs representative memory bandwidth or system power, ensure that your output framebuffer images are of similar visual complexity to your real application use case.

### Avoiding optimistic framebuffer tile elimination

Arm GPUs support an optimization that can skip writing the `COLOR0` framebuffer tile to memory if the content of the tile matches the content already in memory. If your microbenchmark renders the same static image every frame, it could produce very optimistic memory bandwidth data!

If your microbenchmark needs representative memory bandwidth or system power, ensure that your output framebuffer image varies each time the framebuffer is written. That can often be achieved without impacting the cost of the frames by slightly altering the color or camera position.

{{< callout type="info" >}}
Image variation needs to produce tiles that are different to the payload in memory. To avoid tile elimination on an image sourced from an N-buffered swapchain, your rendering needs to differ from the frame rendered N frames ago.
{{< /callout >}}

