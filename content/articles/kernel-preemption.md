---
title: "Is My Linux Kernel Real-Time?"
description: "Inspecting kernel preemption on a Raspberry Pi 4 and distinguishing PREEMPT from PREEMPT_RT."
topics:
  - real-time-linux
  - linux-internals
reading_room: true
---

*Inspecting kernel preemption on a Raspberry Pi 4.*

In our previous experiment, we installed Raspberry Pi OS Lite on a Raspberry Pi 4 and inspected the running Linux system.

One detail in the kernel identification deserves closer attention:

```sh
uname -a
```

Our Raspberry Pi reported:

```text
Linux pi4 6.18.50+rpt-rpi-v8 #1 SMP PREEMPT Debian 1:6.18.50-1+rpt1 (2026-09-11) aarch64 GNU/Linux
```

The word `PREEMPT` suggests that the kernel supports preemption. But does that mean it is a real-time Linux kernel?

In this experiment, we'll examine the kernel configuration and establish exactly what kind of preemption is enabled.

> **Tested configuration:** Raspberry Pi 4 Model B Rev 1.5 (1 GB RAM), Raspberry Pi OS Lite 64-bit, Linux kernel 6.18.50+rpt-rpi-v8.

## 1. What is kernel preemption?

Linux is a multitasking operating system. Many processes compete for CPU time, and the scheduler decides which process runs next.

Preemption allows the scheduler to interrupt a running task and switch the CPU to another task.

However, an important distinction exists between preempting a process executing in user space and preempting code executing inside the kernel.

In a non-preemptible kernel, a task executing kernel code generally continues until it returns to user space, blocks, or reaches another scheduling point.

A preemptible kernel allows scheduling to occur at additional points during kernel execution, reducing the time a higher-priority task may have to wait.

This improves responsiveness, but it does not automatically guarantee predictable timing.

## 2. Inspect the kernel configuration

Linux kernel features are selected through configuration options when the kernel is built.

On our Raspberry Pi, the configuration for the running kernel is available in `/boot`.

We inspected the preemption settings using:

```sh
grep -E '^CONFIG_PREEMPT|^# CONFIG_PREEMPT' /boot/config-$(uname -r)
```

The command searches for configuration lines beginning with `CONFIG_PREEMPT` or its commented-out form.

The expression `$(uname -r)` substitutes the running kernel version, ensuring that we inspect the corresponding configuration file.

Our results included:

```text
# CONFIG_PREEMPT_NONE is not set
# CONFIG_PREEMPT_VOLUNTARY is not set
# CONFIG_PREEMPT_LAZY is not set
CONFIG_PREEMPT=y
CONFIG_PREEMPT_COUNT=y
CONFIG_PREEMPTION=y
# CONFIG_PREEMPT_DYNAMIC is not set
# CONFIG_PREEMPT_RT is not set
```

These settings establish that ordinary kernel preemption is enabled, while full real-time preemption is not.

On some Linux systems, the kernel configuration may instead be available through `/proc/config.gz`, provided the relevant kernel option is enabled.

On our Raspberry Pi, that interface was unavailable, so we used the configuration file in `/boot`.

## 3. PREEMPT versus PREEMPT_RT

The two most important settings in our experiment are:

```text
CONFIG_PREEMPT=y
# CONFIG_PREEMPT_RT is not set
```

They describe two different approaches to kernel preemption.

**CONFIG_PREEMPT**

This enables ordinary kernel preemption. Kernel execution can be interrupted at preemptible points, allowing the scheduler to run a higher-priority task.

However, some kernel operations remain non-preemptible. Interrupt handling, critical sections and other kernel mechanisms can delay scheduling.

**CONFIG_PREEMPT_RT**

PREEMPT_RT extends kernel preemptibility by changing important aspects of interrupt handling and locking.

Among other mechanisms, it moves most interrupt handlers into schedulable threads and converts many kernel locks into sleeping locks with priority inheritance.

These changes reduce the time that high-priority real-time tasks can be prevented from executing.

The objective is not necessarily to make Linux execute instructions faster. It is to make scheduling latency more predictable.

## 4. What does real-time actually mean?

A real-time system is one in which the timing of a result matters, not merely its correctness.

Consider a task that must respond to an external event within 1 millisecond.

If it responds in 100 microseconds on most occasions but occasionally takes 5 milliseconds, it may be unsuitable for the application despite its excellent average performance.

This is the central distinction between **throughput, responsiveness and timing predictability**.

A conventional Linux system may deliver very low average latency while still exhibiting occasional long delays.

A real-time configuration attempts to reduce and control those delays, particularly for high-priority tasks.

PREEMPT_RT is an important part of this approach, but it does not by itself guarantee that an application will meet a particular deadline.

Hardware behaviour, device drivers, interrupt activity, scheduling priorities and application design also matter.

## 5. What have we established?

Our Raspberry Pi is running a preemptible Linux kernel, but not a PREEMPT_RT kernel.

The evidence is explicit:

```text
CONFIG_PREEMPT=y
# CONFIG_PREEMPT_RT is not set
```

This establishes the kernel's configuration, but it does not tell us how much scheduling latency the system actually exhibits.

To answer that question, we need measurements.

In later experiments, we'll investigate timing behaviour using periodic tasks and GPIO signals, measuring the variation between intended and actual execution times.

Eventually, we can repeat the same experiments using a PREEMPT_RT kernel and compare the results.

The important question is no longer simply whether the kernel supports preemption.

**How predictable is its response time under real operating conditions?**
