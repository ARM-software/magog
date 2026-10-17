---
title: Introduction
weight: 1
type: docs
sidebar:
  open: false
---

Graphically rich applications, such as games, are some of the most complex and demanding workloads on mobile devices. The world simulation stresses the CPU, and the rendering workloads push the GPU. Everything must run consistently at interactive frame rates without draining the battery or overheating the device.

Achieving efficient application use of the GPU is not a trivial task. The size of the Vulkan specification — now over 5,300 pages including extensions — gives you some idea of the complexity of modern GPUs. However, it's far from being an impossible task and any developer can learn how to profile and optimize their application to make better use of the graphics stack.

There is a wealth of excellent material available that teaches graphics theory and how to implement graphical techniques. There is surprisingly little material that teaches the practicalities of profiling and optimizing GPU workloads to make them run more efficiently. That is the goal of this book: to teach you what you need to know to effectively profile and optimize your mobile application's GPU workload.

This chapter includes:

* [Book structure][1]: How the book is organized, and what each chapter is designed to provide.
* [What is performance?][2]: What performance means in practice, and the key metrics used to measure it.
* [Managing performance][3]: How to plan, test, and investigate performance during development.

[1]: {{< relref "01_structure.md" >}}
[2]: {{< relref "02_what_is_performance.md" >}}
[3]: {{< relref "03_managing_performance.md" >}}
