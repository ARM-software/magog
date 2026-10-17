---
title: Book structure
type: docs
weight: 1
---

This book is split into chapters, each focusing on a specific functional area, such as geometry or texturing. Each chapter begins with a section explaining the essential hardware background needed for that topic, providing a simple mental model that is useful when interpreting profiling data. It then presents topic-based sections for specific performance issues within that functional area. These sections explain the diagnostic steps used to detect each issue, and the changes you can make to resolve it.

## Arm GPU focus

This book is written from the perspective of a developer profiling and optimizing for Arm® Mali™ and Arm Immortalis™ GPUs.

Some material is specific to the Arm GPU families. Profiling tools in particular are closely tied to the underlying hardware. Other vendors provide their own tools, which may work differently and expose different types of data.

However, much of the optimization advice and many best practices are general-purpose and apply equally well to other GPU architectures. Your application does not need to run on an Arm GPU to benefit from this book.

## Ongoing authoring

This book is being released incrementally, with regular updates as new content becomes available. Check back periodically and read the latest sections.

There is a large amount of graphics API content to cover, and we regularly review and prioritize topics in our backlog. If you have a request, please let us know via the [GitHub issue][GHI] tracker and we will do our best to prioritize it.

## Chapter list and roadmap

The following chapters exist in the current release:

* **[Chapter 1: Introduction][1]** — An overview of the book, including a non-technical introduction to profiling and optimization workflows.
* **[Chapter 2: Mobile Systems][2]** — An introduction to mobile system hardware, and why mobile devices are among the most complex platforms to optimize for.
* **[Chapter 3: Mobile GPUs][3]** — An introduction to mobile GPU hardware, including how it differs from PC and console GPUs, and how GPU profiling differs from CPU profiling.
* **[Appendices][98]** — A collection of short supporting papers that are useful to cover, but that we couldn't find a better home for.

[1]: {{< relref "../01_introduction/_index.md" >}}
[2]: {{< relref "../02_mobile_systems/_index.md" >}}
[3]: {{< relref "../03_mobile_gpus/_index.md" >}}
[98]: {{< relref "../98_appendices/_index.md" >}}
[GHI]: https://github.com/ARM-software/magog/issues
