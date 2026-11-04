---
title: "Linux namespaces: a process with its own view"
description: "Start with a small experiment and see how a process can inhabit a different PID namespace."
label: "LINUX INTERNALS · PRACTICAL LAB"
featured: true
visual: visual-one
book_chapter: "Chapter 8 — Containers"
---

Linux namespaces let processes see different instances of system resources. A PID namespace, for example, gives a process its own process-ID view.

> **Prototype note:** This is an illustrative companion article. Verify and expand the experiment before public release.

## A small experiment

On a Linux host with appropriate privileges, a new PID namespace can be created with `unshare`:

```sh
sudo unshare --pid --fork --mount-proc /bin/sh
ps -ef
exit
```

The shell starts inside a new PID namespace. Its PID numbering differs from the host's view. The `--mount-proc` option arranges a suitable `/proc` view for the experiment.

## Where to go next

Explore the [pid_namespaces(7) manual page](https://man7.org/linux/man-pages/man7/pid_namespaces.7.html) and the book's Containers chapter.
