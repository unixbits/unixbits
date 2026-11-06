---
title: "Setting Up a Raspberry Pi for Linux Experiments"
description: "Preparing a headless Raspberry Pi 4 as a Linux laboratory for kernel, GPIO and real-time experiments."
topics:
  - real-time-linux
  - linux-internals
reading_room: true
weight: 10
---

A Raspberry Pi is a useful platform for exploring Linux internals. It runs a conventional Linux kernel, provides direct access to hardware interfaces, and can be controlled entirely from a remote terminal.

In this experiment, we prepare a Raspberry Pi 4 running the 64-bit Raspberry Pi OS Lite distribution, establish SSH access, and inspect the resulting Linux system.

The objective is to create a reproducible starting point for later experiments involving GPIO, kernel scheduling, timing and real-time behaviour.

> **Tested configuration:** Raspberry Pi 4 Model B Rev 1.5 (1 GB RAM), Raspberry Pi OS Lite 64-bit, Linux kernel 6.18.50+rpt-rpi-v8. Commands and measurements reflect this configuration.

## 1. Hardware and software

Our laboratory consists of:

- Raspberry Pi 4 Model B, revision 1.5, with 1 GB RAM
- 64 GB microSD card
- Raspberry Pi power supply
- Debian Linux workstation
- Local network connection

We use Raspberry Pi OS Lite (64-bit), without a graphical desktop.

The Raspberry Pi operates *headlessly*: no monitor, keyboard or mouse is required.

## 2. Prepare the microSD card

Insert the microSD card into the Linux workstation using a suitable USB reader.

Identify the device:

```sh
lsblk -o NAME,SIZE,MODEL
```

In our case, the relevant device appeared as:

```text
sde    58.2G    MassStorageClass
└─sde1 58.2G
```

The reported capacity of 58.2 GiB is consistent with a nominal 64 GB card.

**Important:** Device names are not fixed. On another computer, the card may appear as `/dev/sdb`, `/dev/sdc` or something else. Identify the correct device carefully before writing an image, as selecting the wrong disk can destroy existing data.

There is no need to format the card beforehand. Raspberry Pi Imager writes the operating-system image, including its partition table and filesystems.

## 3. Install Raspberry Pi OS

On our Debian workstation, we downloaded the Raspberry Pi Imager AppImage:

```sh
wget https://downloads.raspberrypi.com/imager/imager_latest_amd64.AppImage
chmod +x imager_latest_amd64.AppImage
./imager_latest_amd64.AppImage
```

Using Raspberry Pi Imager, we selected:

- Device: Raspberry Pi 4
- Operating system: Raspberry Pi OS Lite (64-bit)
- Storage: 64 GB microSD card
- Hostname: `pi4`
- Username: `pi`
- SSH: Enabled
- Raspberry Pi Connect: Disabled

Network credentials were configured during imaging, allowing the Pi to join the network automatically on its first boot.

After the image was written and verified, we inserted the card into the Raspberry Pi and connected the power supply.

## 4. Establish SSH access

Once the Pi has booted and joined the network, it can be accessed remotely.

From the Linux workstation:

```sh
ssh pi@pi4.local
```

The `.local` hostname uses multicast DNS (mDNS), avoiding the need to discover or configure a fixed IP address.

During our first boot, the intended hostname did not resolve immediately. We inspected the network advertisements using Avahi:

```sh
avahi-browse -art | grep -i pi
```

The Pi initially appeared as `pi4-2.local`. After a subsequent reboot, the expected hostname `pi4.local` became available.

This illustrates a useful practical point: network discovery and hostname resolution may require investigation even when the underlying SSH service is working correctly.

## 5. Configure and update the system

The initial SSH session reported that the workstation's Australian locale was unavailable on the Pi.

We inspected the installed locales:

```sh
locale
locale -a
```

Then configured the required locale:

```sh
sudo dpkg-reconfigure locales
```

We enabled `en_AU.UTF-8` and selected it as the default.

Next, we updated Raspberry Pi OS:

```sh
sudo apt update
sudo apt full-upgrade
sudo reboot
```

After rebooting, we reconnected using SSH.

## 6. Inspect the Linux system

We begin by identifying the running kernel:

```sh
uname -a
```

Our Raspberry Pi reported:

```text
Linux pi4 6.18.50+rpt-rpi-v8 #1 SMP PREEMPT
```

The system runs a 64-bit ARM Linux kernel.

Next, inspect the processor:

```sh
lscpu
```

The relevant characteristics were:

| Property | Observed value |
|---|---|
| Architecture | aarch64 |
| Processor | ARM Cortex-A72 |
| CPU cores | 4 |
| Maximum CPU frequency | 1800 MHz |
| Minimum CPU frequency | 600 MHz |

Memory can be inspected with:

```sh
free -h
```

The Pi reported approximately 905 MiB of usable RAM, consistent with its nominal 1 GB configuration after reserved memory.

We can also identify the board directly:

```sh
cat /sys/firmware/devicetree/base/model
```

Result:

```text
Raspberry Pi 4 Model B Rev 1.5
```

Finally, inspect the SoC temperature:

```sh
cat /sys/class/thermal/thermal_zone0/temp
```

Our measurement was:

```text
41868
```

The value is expressed in millidegrees Celsius, corresponding to approximately **41.9 °C**.

## 7. What have we established?

We now have a working Linux laboratory with:

- A 64-bit ARM Linux system
- Remote SSH access
- A conventional Linux kernel
- Command-line access to system and hardware information
- GPIO interfaces available for subsequent experiments

The system requires no graphical desktop and can be operated entirely from a Linux workstation.

One particularly interesting detail is the kernel's `PREEMPT` designation.

Does that mean the Raspberry Pi is running a real-time Linux kernel?

**Not necessarily.**

In the next article, we'll inspect the kernel configuration and distinguish ordinary kernel preemption from the `PREEMPT_RT` real-time model.
