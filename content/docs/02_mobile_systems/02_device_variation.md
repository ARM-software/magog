---
title: Device variation
weight: 2
type: docs
---

Mobile devices come in a wide range of designs and performance points, and devices in consumers' hands range from the latest premium designs to older mass-market devices. Different classes of device have distinct performance characteristics, and each will show different types of performance problems.

## Different devices, different problems

If you measured the performance of mobile phones used by a large sample of randomly selected consumers, you would see a wide spread. Plotting the results by phone model would give you a distribution like this:

{{< img_tsvg "02/02/device_variation_lt.svg" "02/02/device_variation_dk.svg" "1"  "Consumer device performance distribution"  >}}

### Peak performance is not sustained performance

At the far right of this chart, we have the latest premium devices. These devices are designed for best-in-class performance, using the most efficient CPU and GPU designs, and manufactured using the latest silicon process. These devices have high peak performance and memory bandwidth, but can easily hit thermal issues if applications try to do too much.

### Entry-level devices expose capacity limits

On the left side of this chart are entry-level devices, designed for lower price points. These devices have small CPU and GPU configurations, and often use older {{< term "SoC" >}} designs and silicon process nodes. The smaller configuration means that these devices have much lower performance than high-end devices, so content can easily hit processing, bandwidth, or memory capacity limits. The upside is that these devices rarely suffer from thermal issues; they simply don't have enough processing or bandwidth capacity to cause problematic overheating.

### Mid-range messiness

In the middle of this chart, around the knee of the curve, it is a more complex story.

Some mid-range devices are purpose-built for this market segment. They use smaller configurations than their contemporary premium siblings, but still benefit from the efficiency gains of the latest CPUs, GPUs, and silicon process. These devices have strong performance, and their modern technology means that they are less susceptible to overheating than other mid-range devices.

Other mid-range devices are older premium devices that are still in use, now several years old and sliding down the performance ranking as newer devices overtake them. When first released, these were premium configurations that pushed performance as high as the technology of the time allowed. The combination of older hardware that pushes its limits, and newer content that can more easily reach those limits, makes these devices very susceptible to overheating issues.

## Managing variation

One of the reasons that game consoles are popular with developers is that they provide a consistent development experience. You know exactly what the hardware capabilities are and the performance you are going to get. This makes them a relatively direct platform to develop, optimize, and test against.

Android development is the complete opposite. Your application will need to support multiple generations of devices. These devices can use different GPU designs, GPU vendors, core counts, and frequency configurations. This variability gives consumers a huge amount of choice, but it does make application development much more challenging.

### Defining and testing performance profiles

Most developers we work with review their target audience and define a minimum device specification ({{< term "min-spec" >}}) that they are aiming to support. This defines the baseline graphics API feature set that the application can use, and the baseline performance budget that the application must fit within.

Developers then partition the range of devices above the min-spec into performance profiles. They create an application configuration that uses more advanced graphics to fit each profile's performance budget. Developers typically purchase one or two devices from each profile to use for their testing and profiling.

> [!NOTE]
> It is impossible to test every Android device variation. We recommend that you have enough devices to cover the major GPU vendors and each of their GPU architectures.
>
> Older devices often have more performance quirks, as they ship with less tolerant GPU hardware and use older graphics drivers. We therefore also recommend that you aim to cover the oldest devices that you plan to support. They may have issues beyond their reduction in performance capacity that you will need to handle.

### Managing performance profiles

Performance profiles are not static, and will need adjustment as your application changes after release, or as new devices arrive on the market.

Many developers capture performance telemetry from their released applications, allowing them to verify that all the devices used by their players are performing as expected. Devices that are underperforming can be targeted for optimization, or moved to a lighter-weight profile if they were misclassified.

> [!NOTE]
> Player reviews and feedback are another way to spot devices that are having issues with performance or stability.
