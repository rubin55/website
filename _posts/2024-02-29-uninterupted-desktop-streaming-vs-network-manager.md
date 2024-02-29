---
title: Uninterupted desktop streaming vs. NetworkManager
date: 2024-02-29T12:59:00+02:00
last_modified_at: 2024-02-29T15:27:00+02:00
categories:
  - blog
tags:
  - computers
  - linux
---

For maybe more than a year (maybe two) I've been struggling with getting desktop
streaming (i.e., from a Linux desktop client to a Windows or Linux host system)
working flawlessly. I mean, without interruptions in video frame speed (60fps)
or audio drop-outs.


* TOC
{:toc}

## Uninterupted desktop streaming vs. NetworkManager

Let me begin by describing the experience:

  * Android TV Moonlight or Parsec client to Windows or Linux host: **flawless**
  * Windows Moonlight or Parsec client to Windows or Linux host: **flawless**
  * macOS Moonlight or Parsec client to Windows or Linux host: **flawless**
  * **Linux desktop** using either Moonlight or Parsec: **stuttering, audio and video drop-outs**

I'm streaming 4K at 60fps. I use this for remote desktop work and for gaming.
I observed the above experience starting about two years ago, but I didn't mind
too much because I was still using a MacBook, and a Surface Pro X at the time.

These days I'm fully on Linux, on both my desktops at work and home and on my
laptop, so the issue became unavoidable. Things I've tried and observed:

  * Initially I thought that it had to do with Wi-Fi hardware - I tested with
    cards from Intel, MediaTek, RealTek, even an old Atheros. No dice, stutter
    galore.

  * Any ethernet card, even an 100mbit one, would work flawlessly.

  * I tried different Linux distributions: Pop!_OS, Gentoo, Fedora, Arch Linux.
    No matter, more or less identically configured, all the same behaviour.

  * In `dmesg` you would see a message indicating that either you the client or
    the AP disconnected, and you'd see a reconnection attempt. This aligned with
    when the stuttering was experienced.

### The cause: NetworkManager AP (re)scan

It turns out that NetworkManager (re)scans for APs. I'm not sure if it does that
only for APs serving the same SSID as the network you're on, or more generally
to see if there are other networks available which it might want to connect to,
but I suspect only the former.

In any case, to do that, it needs to drop the connection for a short while, scan
and "quickly" reconnect. In practice this can take more than a second. When you
are streaming 60fps video and audio, that is extremely noticable. On top of that
NetworkManager does this every 20 or 40 seconds or so.

A "workaround" that floats around on the internets is to specify the BSSID field
in the wireless configuration profile within NetworkManager. In my experience
this does not actually work well. If I'd need to quantify it, instead of
experiencing interrupts every minute, it became every 3 or 4 minutes. As I
understand it, NetworkManager parameterizes `wpa_supplicant` in a way that is
essentially not configurable. and instructs it to disconnect, rescan and then
reconnect.

### The solution: systemd-networkd and iwd

Even though I like the convenience of the GUI that NetworkManager enables, Its
shortcomings with regards to stability and performance of the wireless network
connection are too extreme for my use-case.

Since I'm currently on Arch Linux, which uses `systemd`, I opted to forego any
graphical Networking setup and use `systemd-networkd` as a replacement for
NetworkManager and `iwd` as a replacement for `wpa_supplicant`.

I've been using this setup for about a week now, and since then I have zero
network connection interruption issues and can stream 4K at 60fps without issue.

### Quick guide to do this yourself

#### Install `iwd`

(`systemd-networkd` is a part of the `systemd` package).

```shell
# pacman -S iwd
```

#### Turn off and disable NetworkManager and `wpa_supplicant`

```shell
# systemctl disable --now NetworkManager
# systemctl disable --now wpa_supplicant
```

#### Create `systemd-networkd` configuration for device `wlan0`

(your device might be named differently).

```shell
# mkdir -p /etc/systemd/network
# cat <<EOF > "/etc/systemd/network/10-wireless.network"
[Match]
Name=wlan0

[Network]
DHCP=yes
IgnoreCarrierLoss=10s
EOF
```

#### Create `iwd` configuration

```shell
# mkdir -p /etc/iwd
# cat <<EOF > "/etc/iwd/main.conf"
[General]
RoamThreshold=-75
RoamThreshold5G=-80

[Network]
EnableIPv6=false
NameResolvingService=systemd

[Scan]
DisablePeriodicScan=true
EOF
```

#### Turn on and enable `systemd-networkd` and `iwd`

```shell
# systemctl enable --now systemd-networkd
# systemctl enable --now iwd
```

#### Connect to a wireless network

(using device `wlan0` - yours might be named differently).

```shell
# iwctl
[iwd]# station wlan0 scan
[iwd]# station wlan0 connect MY_NETWORK
# (fill in passphrase)
<ctrl-D>
```
