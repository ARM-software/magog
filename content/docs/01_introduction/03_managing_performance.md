---
title: Managing performance
type: docs
weight: 3
---

Performance optimization is sometimes viewed as difficult and reliant on deep expertise. In reality, performance testing is no different from functional testing, and performance investigation is no different from functional debugging. Both require some knowledge of what the system should be doing, and a systematic way to trigger behaviors that show that it's actually doing what you expect.

Difficulties arise when you have no intuition about what your performance should be, or how changes are expected to impact performance. Without that intuition, you have no direction and are just trying things blindly.

## Device planning

Mobile application business models often depend on achieving a sizable user base. This means you must balance the device minimum specification {{< term "min-spec" >}} that you are targeting, and the number of potential users that implies.

Before starting a project, we recommend that you buy some devices that are representative of your min-spec. Prototype the major components of your renderer on them to determine what performance is achievable. You should repeat this for any other devices that you want to target with a higher-quality performance profile.

## Performance budgets

At this early phase you have some of the big budget choices to make:

* What frame rate do you want to target?
* What resolution do you want to target?
* What color formats do you want to target?

These are important to get right because, when combining all three together, you get N<sup>2</sup> scaling for processing cost and N<sup>3</sup> scaling for memory bandwidth.

Consider an application moving a target from 720p30 (720p, 30 FPS) and Low Dynamic Range 8-bit color ({{< term "LDR" >}}), to 1080p60 and High Dynamic Range {{< term "FP16" >}} color ({{< term "HDR" >}}).

The original cost of a single render pass is:

* **Processing:** (1280 * 720) * 30 = 27.6M pixels/second.
* **Bandwidth:** (1280 * 720) * 4 * 30 =  110.5M bytes/second.

The cost of an enhanced single render pass is:

* **Processing:** (1920 * 1080) * 60 = 124.4M pixels/second.
* **Bandwidth:** (1920 * 1080) * 8 * 60 =  995.3 bytes/second.

The increased target means that the device will need to handle 4.5 times more processing and 9 times more memory bandwidth.

One useful tool that we use with developers is setting approximate resolutions and performance budgets for the different parts of their rendering pipeline. This process can start by assigning a time budget to each phase, aiming to use only 90% of the total available frame period to keep some headroom. The example below shows a breakdown for an application targeting 1080p60.

| Pass           | Resolution | Budget (ms) |
| :---           |       ---: |        ---: |
| Shadow map     | 1024x1024  |         2.0 |
| G-buffer       | 1920x1080  |         4.0 |
| Lighting       | 1920x1080  |         6.0 |
| Motion blur    | 1920x1080  |         2.0 |
| User interface | 1920x1080  |         1.0 |

Based on knowledge of the target device, we can turn this time estimate into a budget for the number of GPU shader cycles per pass and per pixel. Our example application is targeting an Arm® Mali™-G610, with 2 shader cores running at 800 MHz.

| Pass           | Resolution | Budget (cy/pass) | Budget (cy/px) |
| :---           |       ---: |             ---: |           ---: |
| Shadow map     | 1024x1024  |            3.2 M |           3.05 |
| G-buffer       | 1920x1080  |            6.4 M |           3.09 |
| Lighting       | 1920x1080  |            9.6 M |           4.63 |
| Motion blur    | 1920x1080  |            3.2 M |           1.54 |
| User interface | 1920x1080  |            1.6 M |           0.77 |

What should be immediately obvious is that, although the cycle budgets per pass are quite large, the cycle budget per pixel is small. When you have millions of shader threads to run, you do not get many shader cycles per thread.

Now that you have a per-thread budget, you can start to determine if your shader algorithms can fit in this budget. You don't even need hardware to prototype this level of analysis, although it will give more accurate data if you do. The [Mali Offline Compiler][1] can give a shader cost estimate for any recent Arm GPU, without needing a physical device.

Compare the shader cost estimate against the estimated budget. This tells you whether your chosen algorithms are feasible for the target GPU at the resolution and frame rate you want. For graphics passes, note that although we computed the budget as "per pixel", it must also cover the cost of any geometry processing needed.

> [!NOTE] What can a GPU do in one cycle?
> GPUs are high-performance accelerators, so even one shader core cycle can do a huge amount of work. A Mali G1 shader core can do 128 {{< term "FP32" >}} {{< term "FMA" >}}s, 8 bilinear texture lookups, 8 fragment blends, and 4 output pixel writes per cycle!
>
> The [Arm GPU datasheet][2] gives per-core performance throughput for all recent Arm GPUs.

## Performance testing

Once you have a performance budget, you need workloads that let you measure your current performance. You can then compare those measurements against the budget.

The games industry still relies very heavily on manual play testing, with a QA team playing the game and reporting issues. This definitely has a place, but for day-to-day performance management we advocate using repeatable tests that can be automated.

### Repeatability

The first problem with testing performance just by playing the game is that no two test runs are exactly the same. The player will move differently, and in-game events will trigger at slightly different times. When you optimize something, you want a scenario that can generate the same rendering sequence twice. The only difference should be caused by the optimization you are testing. Many optimizations are small, so even minor changes in the test workload could mask the impact of the optimization.

The second problem is that just playing the game will often result in observing typical behavior. It may not systematically find the complex scenarios where performance will be stressed. Synthetic test scenes are a much more convenient and reliable way to cover stressful scenarios so that you can be sure that your target devices can keep up.

There are two common approaches to implementing repeatable test scenarios:

* Fixed duration scenarios
* Fixed frame count scenarios

<!-- TODO: This would benefit from some simple images -->

Fixed duration scenarios render the test scene for a fixed number of seconds, animating each frame based on the elapsed time since the last frame. The number of frames rendered in each test run varies depending on the performance of the device. This is useful if you want demo scenes that animate normally when viewed. However, it means that you can't do exact frame-to-frame comparison over two runs because frame content depends on performance.

Fixed frame count scenarios render the test scene for a fixed number of frames, animating each frame using a constant timestep that is independent of device performance. The test duration varies depending on the performance of the device. When viewing the scene on a device, the animation speed may appear to slow down and speed up as frame complexity varies. The big advantage of this approach is that a specific frame will render the same thing every time you run the test. This lets you easily reproduce issues and measure the impact of changes.

## Automation

Initially you may run performance tests manually. However, automating the bulk of your performance testing can be a worthwhile investment, even if you still do occasional manual performance reviews.

Automated testing is more reliable than manual testing, because computers will always repeat the same sequence of steps. It is also more scalable, because you can run tests on more devices than a human could run manually. This is particularly important for Android developers, given the number of device configurations on the market.

When automating performance testing, do not just track simple pass/fail results that fail if a test exceeds budget. Extract the underlying performance metrics, store them in a database, and have a dashboard with graphs so that you can monitor your performance trends over time. This will let you detect trending slowdowns caused by an accumulation of small regressions before they become a user-visible performance drop.

## Performance investigation

Performance testing is an essential part of any project, and works best when automated. However, it is a defensive technique that aims to stop things breaking, not a proactive tool for making something better. No matter how good your automated testing is, there will always be a need for human-driven performance investigation. A developer, armed with a debugger and profiling tools, can investigate causes of problems highlighted by tests and identify new areas that could be optimized.

Application performance reviews analyze the latest game build and present the results to the wider team. They are a great way to ensure that everyone is aware of current performance and any issues that need to be fixed. Most studios we work with run these reviews regularly during development, and for any post-release content updates.

## Performance optimization

Once you have investigated and have some ideas about what needs to be optimized, you can gather more detailed performance data and start developing optimizations. This can seem daunting, but applying some systematic methodology helps.

### The scientific method

Optimization is a data-driven exercise, and can therefore follow the scientific method. Based on data and analysis from earlier investigations, you can:

* Form a hypothesis about why part of a frame is slow.
* Design an experiment to test that hypothesis.
* Predict what your experiment will change.
* Measure the result.
* Compare evidence against the prediction.
* Repeat.

This is another situation where any earlier test automation helps. Automation can run performance tests on candidate patches and compare measured performance against the per-patch baseline. This makes it easy to run performance experiments across a range of devices.

{{< callout type="info" >}}
Early experiments do not need to be production-ready, and can focus on showing that the application performance is sensitive to the area that you changed. If no performance change was observed, your starting hypothesis might be incorrect.
{{< /callout >}}

[1]: https://developer.arm.com/Tools%20and%20Software/Mali%20Offline%20Compiler
[2]: https://developer.arm.com/documentation/102849/latest/
