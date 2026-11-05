---
topics:
  - linux-internals
title: "Linux namespaces: a process with its own view"
description: "Explore PID namespaces with unshare, compare process IDs from inside and outside, and discover why /proc matters."
label: "LINUX INTERNALS · PRACTICAL LAB"
featured: true
visual: visual-one
book_chapter: "Chapter 8 — Containers"
---

A Linux process normally sees the host's process-ID space: PID 1, other users' processes (subject to permissions), and the processes it starts itself. **PID namespaces** allow a group of processes to see a different numbering of processes, beginning with their own PID 1.

This is one of the mechanisms used by Linux containers. It does **not** create another kernel. Processes in the namespace still run on the same Linux kernel as the host.

## What is a namespace?

A namespace changes which instance of a particular system resource a process sees. Linux provides several namespace types, including:

- **PID:** process IDs and the visible process hierarchy.
- **Mount:** the set of filesystem mounts.
- **Network:** network devices, addresses, routing tables and ports.
- **UTS:** hostname and NIS domain name.
- **IPC:** certain interprocess communication resources.
- **User:** mappings of user and group IDs, and associated capabilities.
- **Cgroup:** the view of cgroup paths.
- **Time:** selected clock offsets.

Namespaces isolate *views* of resources; they are not, by themselves, a complete security boundary or a full container. Container runtimes combine namespaces with other facilities such as cgroups, filesystem setup and security controls.

## Experiment: create a PID namespace

This experiment uses `unshare`, a standard Linux utility. Run it on a test machine or VM where you have administrator access. You will need `ps` (usually supplied by `procps`).

First, on the host, inspect the current shell and PID 1:

```sh
printf 'Host shell PID: %s\n' "$$"
ps -p 1 -o pid,ppid,comm
```

Now create a new PID namespace, along with a private mount namespace for the new `/proc` mount:

```sh
sudo unshare --pid --fork --mount --mount-proc /bin/sh
```

Inside the new shell, run:

```sh
printf 'Namespace shell PID: %s\n' "$$"
ps -p 1 -o pid,ppid,comm
ps -ef
```

The new shell should report **PID 1**. The `ps` output should show a much smaller process population than on the host: principally the shell and the commands it starts. Exit when finished:

```sh
exit
```

The precise process names and numbers other than PID 1 will vary. If `unshare` reports `Operation not permitted`, the host's security policy or environment may disallow creating the required namespaces; try an appropriate test VM rather than changing security settings just for this exercise.

## Why `--fork` and `--mount-proc`?

The `--pid` option creates a new PID namespace for **children** of the calling process. It does not retroactively move the calling process into that namespace. `--fork` starts a child so that `/bin/sh` actually runs inside the new PID namespace.

There is a second subtlety: `ps` usually reads process information from `/proc`. A new PID namespace alone does not automatically give the shell a matching `/proc` mount. The `--mount-proc` option mounts a `/proc` instance appropriate to the new PID namespace, so tools such as `ps` show the namespace's process view.

We also use `--mount` to create a separate mount namespace. This keeps the experimental `/proc` mount changes away from the host's mount namespace. On typical util-linux versions, `--mount-proc` already implies a mount namespace; spelling out `--mount` makes the intention explicit.

## One process, two PIDs

The same process can have different PID numbers depending on where it is observed. A process inside a nested PID namespace has a PID in that namespace **and** a PID in its ancestor namespace. The host can see processes in its descendant PID namespaces, but processes inside the child namespace cannot see arbitrary host processes through their namespace's PID view.

To observe both perspectives, open **two terminals**. In terminal A, start the namespace and leave the shell running:

```sh
sudo unshare --pid --fork --mount --mount-proc /bin/sh
printf 'Inside PID: %s\n' "$$"
sleep 300 &
```

The shell reports PID 1, while `sleep` receives another small PID. In terminal B, on the host, locate the `sleep 300` process:

```sh
ps -eo pid,ppid,args | grep '[s]leep 300'
```

Its host PID will generally be different from the PID shown inside the namespace. To inspect the mapping, substitute the **host PID** in:

```sh
grep '^NSpid:' /proc/HOST_PID/status
```

For a process in a single nested PID namespace, `NSpid` commonly lists two numbers: the PID in the host's namespace followed by the PID in the child namespace. If multiple `sleep 300` processes exist, identify the correct one before comparing. Stop the experiment in terminal A with `exit`; the background process may be terminated when PID 1 exits.

## Why PID 1 is special

In an ordinary Linux system, PID 1 is the initial userspace process. In a PID namespace, the first child process created in that namespace becomes its **init process**, also numbered PID 1.

This matters because PID 1 has special responsibilities and signal semantics. In particular, it should reap orphaned child processes. When namespace PID 1 terminates, the kernel terminates the other processes remaining in that PID namespace. A real container runtime therefore needs to think carefully about which process runs as PID 1 and how it handles signals and child processes.

## What the experiment demonstrates

The experiment has changed the process-ID view, not the underlying kernel. The namespace shell is still scheduled by the host kernel, uses the same system-call interface, and competes for physical resources unless other mechanisms impose limits.

This distinction is central to understanding containers: **namespaces control visibility and isolation of selected resources; cgroups control accounting and limits for groups of processes.** A container is an arrangement of several Linux facilities, not a miniature virtual machine.

## Where to go next

For the kernel's detailed rules, read [pid_namespaces(7)](https://man7.org/linux/man-pages/man7/pid_namespaces.7.html), [namespaces(7)](https://man7.org/linux/man-pages/man7/namespaces.7.html) and [unshare(1)](https://man7.org/linux/man-pages/man1/unshare.1.html).

For the broader picture, see **Chapter 8 — Containers** in *Understanding Linux: The Kernel Perspective*, which places namespaces alongside the other mechanisms used to construct Linux containers.
