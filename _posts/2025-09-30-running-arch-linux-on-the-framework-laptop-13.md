---
title: Running Arch Linux on the Framework Laptop 13
date: 2025-09-30T11:02:00+02:00
last_modified_at: 2025-09-30T11:02:00+02:00
categories:
  - blog
tags:
  - computers
  - linux
---

This article sums up why and how I run Arch Linux on
[my Framework Laptop 13](/blog/why-i-am-getting-a-framework/), which I received
on the 3rd of July 2023, and am still as of today happily using!

I'm *extremely* impressed and very happy with this laptop - it is by far one of
the best devices I've owned in a long time. It is near-silent, It uses ~5watts
at idle, it's fast, it's sturdy and it looks good!

This is going to be quite a long article, so here's a Table of Contents:

* TOC
{:toc}

## Why Arch Linux

Prior to my Framework Laptop adventures, I've been planning to move to Arch
Linux for a while. I'm a long-time Linux desktop user. I started out with those
Red Hat CD-ROMs you'd buy at your local bookshop (this was around '96). I
fooled around with SCO UnixWare, had a whole period of SGI IRIX after that and
then some distro-hopping to Debian, Fedora, Arch Linux (in 2006), Gentoo and
Void Linux, more-or-less in that order.

The last two-ish years I've been running on Void Linux, which I still strongly
recommend and love - it's a really good distribution with a nice balance between
stability and simplicity.

I used the OS package manager for the essentials, like Gnome, Firefox, the
terminal, Wayland and X11, and used `/opt` like a sort of `Program Files` or
`Applications` directory where I had my own programs like IntelliJ, Postman,
PostgreSQL, etc.

This works relatively well: you don't have a lot of demands on package
availability in the distro itself and you keep things stable - only update your
own stuff when you feel like it and/or when you need to.

At a certain point I had about 70 applications in `/opt` which became a burden
to keep up to date, so I set out to find a distribution that packages as much
as possible of the software I use, and where packaging it yourself is as simple
as possible.

I definitely prefer a rolling-release distro. I checked out Nix, Gentoo and Arch
Linux. Nix I found really appealing (hard declarative) but I hated the syntax
(maybe Guix one day? I like Scheme better).

Gentoo did have a comparable set of packaged software available, and it *is*
one of my favorite distributions, but I had too many issues on my test-build
virtual machine which blocked me from experimenting prior to running it on my
daily driver.

So why move to Arch? Well, in one word: mind-share. The wiki, the amount of
packaged software, sane package standards:

  * The Arch wiki has essentially become the de-facto standard Linux wiki
  * Really huge package collection, fantastic community participation with AUR
  * Upstream stable usually means an upstream update is in repos within minutes
  * Does not have separate dev/devel packages for headers

The above three things means a lot of convenience to a Linux desktop user. On
Arch Linux, 99.9% of the software I ever used on Linux is packaged, most of it
in the official repositories and a few in AUR. Next to that, AUR and PKGBUILD
are so easy to get into, so you can easily package things yourself and share it
with the community.

On most other distros the gap between what's packaged and what isn't is quite
a bit larger which means you need alternative ways to get that software and keep
it up-to-date, which in practice becomes error-prone and time-consuming.

On Arch Linux I made the choice to run *everything* packaged with `pacman`. If
I need something that isn't packaged yet (currently the case for 5 packages of
which I packaged one already), I package it and publish it to AUR.

The rest of this article is essentially a how-to with notes about what I run
and install. Feel free to peruse and/or replay!

## BIOS settings

You can find the [BIOS guide](https://community.frame.work/t/bios-guide/4178)
on the Framework community website. The only settings I changed were:

  * CPU Configuration -> Boot Performance mode: **MAX BATTERY**
  * CPU Configuration -> Intel Turbo Boost Max Technology 3.0: **DISABLED**
  * Secure Boot -> Enforce Secure Boot: **DISABLED**

The above causes the laptop to effectively run in a lower TDP setting. I think
it goes from 28watts to 22watts. This has a dramatic effect on the battery life
and thermals of the device. I don't want a wild beast that blasts the fans as
soon as I type `ls`, so these settings are great for me. Did I mention the
laptop performs really great with these settings?

With the above and the settings I configure in TLP I get ~3watts idle with
screen on normal brightness, Wi-Fi and Bluetooth enabled, and about ~45 celsius
core temperatures.

## Basic installation

Basically, [download](https://archlinux.org/download/) the ISO and follow the
[installation guide](https://wiki.archlinux.org/title/Installation_guide). I
opted to use `systemd-boot` as boot manager and either `systemd-networkd` or
`networkmanager-iwd` as my network configuration tooling.

### Additional kernel parameters I use

Whichever boot manager you use, you might want to set a few extra kernel
parameters. I found the following additionals handle a bunch of stuff nicely
on my Framework Laptop 13:

```
acpi_mask_gpe=0x1A acpi_osi="!Windows 2020" libata.allow_tpm=1 mem_sleep_default=deep net.ifnames=0 nvme.noacpi=1 nvme_core.default_ps_max_latency_us=0 root=LABEL=LINUX rtc_cmos.use_acpi_alarm=1 rw split_lock_detect=off tpm_tis.interrupts=0
```

## Packages I install

Next to the default `core` and `extra` repositories, I enable `multilib` in
`pacman.conf` before installing. Additionally, I blacklist `pam_systemd_home.so`
because I don't use `systemd-homed` and it spams the journal on any login or
auth action:

```diff
--- pacman.conf.orig    2023-08-06 12:09:44.849855338 +0300
+++ pacman.conf 2023-08-06 12:01:30.261984035 +0300
@@ -26,7 +26,7 @@
 #IgnoreGroup =

 #NoUpgrade   =
-#NoExtract   =
+NoExtract    = usr/lib/security/pam_systemd_home.so

 # Misc options
 #UseSyslog
@@ -87,12 +87,11 @@
 #[multilib-testing]
 #Include = /etc/pacman.d/mirrorlist

-#[multilib]
-#Include = /etc/pacman.d/mirrorlist
+[multilib]
+Include = /etc/pacman.d/mirrorlist
```

You can install the following packages right after your system comes up first
boot, or you could do it during install with `pacstrap` (it doesn't really
matter, but in any case, make sure `core`, `contrib` and `multilib` are enabled
in `/etc/pacman.conf` first):

```shell
pacman -S --needed acpi acpica acpi_call-dkms acpid alacritty alsa-firmware alsa-tools alsa-ucm-conf alsa-utils ansible-language-server ansible-lint ant antlr4 ardour aribb25 arj autoconf automake aws-cli base-devel bash bash-language-server bc bear bind biome bison blender bower btop bubblewrap busted bustle calibre cameractrls carla cdemu-client cdemu-daemon cdrdao cdrtools cifs-utils clang clinfo clojure cmake corkscrew cpio cppcheck cpupower ctags cue cups curl dagger dash dconf-editor dcraw ddcutil debugedit delve deno desmume devtools direnv discord distrobox dive dmidecode dmraid dnsmasq docker docker-buildx docker-compose dool dos2unix dosfstools doxygen d-spy dvd+rw-tools dvisvgm editorconfig-checker editorconfig-core-c efibootmgr elixir emacs-wayland erlang eslint ethtool exfatprogs extra-cmake-modules fakeroot fastfetch fd fdupes file file-roller fio firefox flex fltk1.3 fontforge foomatic-db-engine foomatic-db-nonfree-ppds foomatic-db-ppds fractal freetype2-demos fs-uae fs-uae-launcher furnace fwupd fzf gamemode gamescope gcc gdb gdm gemini-cli ghidra gimp git git-filter-repo github-cli git-lfs glab gnome-backgrounds gnome-browser-connector gnome-calculator gnome-characters gnome-control-center gnome-disk-utility gnome-logs gnome-session gnome-settings-daemon gnome-shell gnome-shell-extension-appindicator gnome-shell-extension-desktop-icons-ng gnome-shell-extensions gnome-system-monitor gnome-themes-extra gnome-tweaks gnupg gnuplot go gopls go-tools gparted gperf gradle groovy gtksourceview3 gtkwave guile gvfs gvfs-gphoto2 gvfs-mtp gvfs-nfs gvfs-smb handbrake harfbuzz-cairo hdparm helix helm hplip htop hunspell hunspell-de hunspell-en_gb hunspell-en_us hunspell-es_es hunspell-fr hunspell-nl i2c-tools ifuse img2pdf inetutils inkscape iperf3 iptables-nft irssi iverilog iwd jekyll jfsutils jq jupyterlab jupyterlab-lsp jupyter-lsp jupyter-notebook just k9s kafka kddockwidgets kotlin kubectl kubeseal leiningen lhasa libblockdev-crypto libblockdev-dm libblockdev-fs libblockdev-loop libblockdev-lvm libblockdev-mdraid libblockdev-mpath libblockdev-nvme libblockdev-part libblockdev-swap libebur128 libgoom2 libindicator-gtk3 librecad libreoffice-still libretro-beetle-pce libretro-beetle-psx-hw libretro-blastem libretro-core-info libretro-desmume libretro-dolphin libretro-flycast libretro-mame libretro-mgba libretro-mupen64plus-next libretro-nestopia libretro-picodrive libretro-ppsspp libretro-sameboy libretro-scummvm libretro-snes9x libtiger libva-mesa-driver libva-utils libvirt libwebp-utils libxcrypt-compat linux-firmware-bnx2x linux-firmware-liquidio linux-firmware-mellanox linux-firmware-nfp linux-firmware-qlogic linux-headers live-media lld lldb lm_sensors loupe lshw lsof lsscsi ltrace lua-language-server make mame mame-tools man-db mangohud man-pages mattermost-desktop maven mbedtls2 mednafen mesa-demos mesa-utils mesa-vdpau meson mgba-qt minikube mitmproxy mono mono-msbuild moreutils mpv mtools multipath-tools mupdf-tools mupen64plus mutter nasm nautilus neovide neovim netbeans networkmanager-openvpn networkmanager-vpn-plugin-openvpn net-tools nfs-utils ninja nmap nodejs noto-fonts-emoji npm ntfs-3g nuget nvchecker nvme-cli nvtop ollama openai-codex openbsd-netcat opencl-clhpp opencl-headers opencv openldap openssh openvpn osv-scanner p7zip pacman-contrib pacutils papers papirus-icon-theme patch patchelf pciutils pdfarranger perf perl perl-lwp-protocol-https perl-net-dbus perl-x11-protocol perl-yaml pinentry piper pipewire pipewire-alsa pipewire-jack pipewire-pulse pipewire-v4l2 pkgconf postgresql powertop ppsspp prettier projectm protonmail-bridge psutils python python-jsbeautifier python-jwcrypto python-kubernetes python-ldap python-lsp-server python-nose python-numpy python-opencv python-opengl python-patch-ng python-pip python-pycryptodomex python-pylint python-pyopenssl python-pytest python-rope python-setuptools python-sphinx python-sphinx-autobuild python-sympy python-websockets python-wheel qbittorrent qemu-full qjackctl qpwgraph qt5ct qt5-declarative qt5-tools qt5-wayland qt6ct qt6-multimedia-ffmpeg qt6-tools qt6-wayland quodlibet rabbitmq racket ranger realtime-privileges rebuild-detector recode retroarch retroarch-assets-glui retroarch-assets-ozone ripgrep rlwrap rsync ruby ruby-base64 ruby-bigdecimal ruby-csv ruby-default-gems ruby-irb ruby-rake-compiler rustup samba sane sbt scons screen scummvm sdcc sdl2_mixer seahorse seatd shfmt signal-desktop simple-scan smartmontools smbclient snapshot snes9x-gtk sof-firmware sox speedtest-cli squashfs-tools steam step-ca step-cli stern strace s-tui stylelint sudo syncthing sysdig sysprof sysstat tailwindcss-language-server taplo-cli tar terminus-font texinfo texlab texlive-basic texlive-bin texlive-fontsrecommended texlive-latex texlive-latexextra texlive-latexrecommended texlive-pictures the_silver_searcher thunderbird tidy tinyxxd tmux tokei traceroute tracker3-miners tree tree-sitter-cli tree-sitter-grammars ttf-ibm-plex ttf-ubuntu-font-family turbostat twine typescript typescript-language-server udftools udisks2-lvm2 uncrustify unixodbc unzip urlwatch usbutils util-linux uucp valgrind valkey vdpauinfo vhba-module-dkms virt-manager vkd3d vscode-css-languageserver vscode-html-languageserver vscode-json-languageserver vue-language-server vulkan-tools w3m wayland-utils wgetpaste whois wimlib wine wine-mono wireless-regdb wireless_tools wireplumber wireshark-cli wireshark-qt wl-clipboard wmctrl wol xcb-util-errors xchm xclip xdelta3 xdg-desktop-portal-gnome xdg-user-dirs-gtk xdg-utils xdotool xfsprogs xorg-fonts-100dpi xorg-fonts-misc xorg-font-util xorg-mkfontscale xorg-server xorg-server-devel xorg-xauth xorg-xdpyinfo xorg-xdriinfo xorg-xev xorg-xfontsel xorg-xhost xorg-xinit xorg-xinput xorg-xkill xorg-xlsclients xorg-xprop xorg-xrandr xorg-xrdb xorg-xset xorg-xsetroot xorg-xvinfo xorg-xwayland xorg-xwininfo yarn yasm yq yt-dlp zig zip zls zstd
```

### Remove if you don't want brltty

The `qemu-full` package depends on `qemu-chardev-baum` which in turn pulls in
`brltty`, a braille (virtual?) keyboard service. If you don't need that:

```shell
pacman -R qemu-chardev-baum qemu-full brltty
userdel brltty
```

### Additionally install when you want dynamic application of power management settings

You can install `tlp` to enable dynamically applying power management settings,
based on if your power connector is connected or if you're on battery. We also
install `tlp-pd` which is the TLP project's DBUS interface compatible
implementation of `power-profile-daemon` (which itself conflicts with `tlp`).
Installing `tlp-pd` Makes it possible to control TLP from things like Gnome,
Cosmic, KDE, etc.

```shell
pacman -S --needed tlp tlp-pd
systemctl enable --now tlp-pd
```

### Additionally install when you want to use dracut instead of mkinitcpio

I use `dracut` on my desktop computer, because I need working LVM RAID mirrors
there. I couldn't get that to work with `mkinitcpio` based initrd.

If you want to use `dracut` instead of `mkinitcpio` to generate initrd images,
you need to install it and remove the default initrd tooling:

```shell
pacman -S --needed dracut
pacman -R mkinitcpio mkinitcpio-busybox
```

Note that there are no hooks by default for rebuilding dracut-based initrd
images, so you'd need to do that manually after `linux` kernel package updates.
For example:

```shell
# Kernel version in package contains dot, on-disk it contains dash, hence the sed.
export KERNEL_VERSION=$(pacman -Q linux | awk '{print $2}' | sed 's|\.arch|-arch|g')
cp /lib/modules/$KERNEL_VERSION/vmlinuz /boot/vmlinuz-linux
dracut --kver $KERNEL_VERSION --force /boot/initramfs-linux-fallback.img
dracut --hostonly --no-hostonly-cmdline --kver $KERNEL_VERSION --force /boot/initramfs-linux.img
```

### Additionally install when you use X11 instead of Wayland and want gestures

Wayland provides working gestures on Gnome out of the box, and Wayland is the
default on Arch Linux. If, however, you have some reason to use X11 instead of
Wayland by default and you want gestures to work, you can install Touchégg:

```shell
pacman -S --needed touchegg
```

Make sure you enable the service:

```shell
systemctl enable --now touchegg
```

And also install the required Gnome [extension](https://github.com/JoseExposito/gnome-shell-extension-x11gestures).


### Additionally install on devices with fingerprint reader

Devices like the Framework Laptop 13 have a fingerprint reader. If you want to
use it, make sure to install the following packages:

```shell
pacman -S --needed libfprint fprintd
```

And make sure you enable the service:

```shell
systemctl enable --now fprintd
```

To configure the fingerprint reader, I needed to upgrade the firmware. Version
`01000320` is known to not work with Linux. In my case, I had to use an older
version of `fwupd` (version 1.9.5 to be specific) - 1.9.10 did not work, but
I've had a report recently (thanks, Jan Schoone) that `fwupd` version 1.9.13
and possibly higher, does actually work these days (your milage may vary, etc).

In any case, I followed the instructions [here](https://knowledgebase.frame.work/en_us/updating-fingerprint-reader-firmware-on-linux-for-13th-gen-and-amd-ryzen-7040-series-laptops-HJrvxv_za)
to update to the required firmware version `01000330` to make the fingerprint
reader work with Linux. Instructions below are based on the linked document:

```shell
# First downgrade fwupd...
wget --continue https://archive.archlinux.org/packages/f/fwupd/fwupd-1.9.5-2-x86_64.pkg.tar.zst --output fwupd-1.9.5-2-x86_64.pkg.tar.zst
pacman -U fwupd-1.9.5-2-x86_64.pkg.tar.zst
# Get firmware file and install...
wget --continue https://github.com/FrameworkComputer/linux-docs/raw/main/goodix-moc-609c-v01000330.cab --output goodix-moc-609c-v01000330.cab
fwupdtool install --allow-reinstall --allow-older goodix-moc-609c-v01000330.cab
```

Note: There might be a transfer error mentioned at the end. This can be safely
ignored. Reboot and start a fresh terminal and show the device:

```shell
fwupdmgr get-devices 1e8c8470-a49c-571a-82fd-19c9fa32b8c3
```

Note: The above command sometimes times out the first time it's run. Just run it
again and you'll see some details about the fingerprint reader. Notably, the
version should now be: `01000330`. You can now enroll fingerprints.

If your fingerprint reader is working, you can continue to follow the steps
[here](https://wiki.archlinux.org/title/Fprint) to set it up for usage.

### Additionally install on devices with Intel graphics

My Framework Laptop 13 is intel-based, so I install these packages additionally:

```shell
pacman -S --needed intel-gpu-tools vulkan-intel intel-media-driver libvdpau-va-gl intel-graphics-compiler intel-compute-runtime gst-plugin-va
```

Also, from AUR, I install these, making sure their version is synced with the
above mentioned 64-bit versions, These make it possible for 32-bit apps to also
use VAAPI-based hardware acceleration:

```
lib32-intel-gmmlib
lib32-intel-media-driver
```

### Additionally install on devices with AMD graphics

If you have an AMD device instead, you might want these ones:

```shell
pacman -S --needed radeontop vulkan-radeon gst-plugin-va ollama-rocm
```

### Additionally install on devices with NVidia graphics

If you have an NVidia device instead, you might want these ones:

```shell
pacman -S --needed cuda ffnvcodec-headers libva-nvidia-driver nvidia-cg-toolkit nvidia-settings nvidia-utils nvidia-open-dkms nvtop opencl-nvidia openssl-1.1 gst-plugins-bad ollama-cuda opencv-cuda python-opencv-cuda nsight-compute nsight-systems
```

Note: The `openssl-1.1` package is a (missing) dependency for
`libnvidia-pkcs11.so` contained in `nvidia-utils`.

Note 2: The `opencv-cuda` and `python-opencv-cuda` packages replace the `opencv`
and `python-opencv` packages installed previously.

### Installing official sublime-text and sublime-merge packages

[Sublime Text](https://www.sublimetext.com/) has an official Arch linux
repository for its `sublime-text` and `sublime-merge` packages. If you want to
use those, do the following (as root):

```shell
# Install and sign the sublimehq pubkey.
curl -O https://download.sublimetext.com/sublimehq-pub.gpg
pacman-key --add sublimehq-pub.gpg
pacman-key --lsign-key 8A8F901A
rm sublimehq-pub.gpg

# Add sublime-text package repository to pacman.conf.
cat <<EOF >> "/etc/pacman.conf"
# Official Sublime Text and Sublime Merge repository.
[sublime-text]
SigLevel = Required DatabaseRequired TrustedOnly
Server = https://download.sublimetext.com/arch/stable/x86_64
EOF

# Update local pacman databases and install.
pacman -Syu sublime-text sublime-merge
```

The above is based on [these instructions](https://www.sublimetext.com/docs/linux_repositories.html#pacman).

### Installing lib32 package equivalents (optional)

I like to install the lib32-* package equivalents of packages I installed. There
isn't an easy way to do this and it is a bit messy, but here's how I do it:

```shell
# Clean beginnings.
mkdir -p ~/Desktop
rm ~/Desktop/lib32-candidates ~/Desktop/lib32-notfound

# Append a list of possible lib32-* package names by prepending to package name.
for p in `pacman -Qq | grep -v lib32`; do echo lib32-$p >> ~/Desktop/lib32-candidates; done

# Append a list of possible lib32-* package names which consist of lib32-firstnamepart.
for p in `pacman -Qq | grep -v lib32 | cut -d- -f1 | sort -u`; do echo lib32-$p >> ~/Desktop/lib32-candidates; done

# Remove known-not-working elements and make sure the output file is sorted and unique.
cat ~/Desktop/lib32-candidates | grep -v rustup | grep -v openssl-1.1 | grep -v mesa-amber | grep -v mesa-demos | sort -u -o ~/Desktop/lib32-candidates

# Abuse pacman -S to obtain invalid package names.
pacman -S --needed `cat ~/Desktop/lib32-candidates | sort -u` 2>&1 | grep 'error: target not found' | awk '{print $5}' > ~/Desktop/lib32-notfound

# Make sure that's sorted and unique too.
sort -o ~/Desktop/lib32-notfound ~/Desktop/lib32-notfound

# Use comm to diff the lists and only feed valid package names to pacman -S.
pacman -S --needed `comm -23 ~/Desktop/lib32-candidates ~/Desktop/lib32-notfound`
```

## Configure networking

You can configure networking in all kinds of ways, but the two I document are
`networkmanager-iwd` and `systemd-networkd`.

I specifically chose `networkmanager-iwd`, because I have had extensive
problems with `wpa_supplicant` in combination with high performance network
requirements, like when you need to stream a 4K desktop at 60fps. Due to how
`wpa_supplicant` is instrumented by NetworkManager, that kind of use-case is
virtually impossible to get right. `iwd` seems to handle this use-case much
better and so I opt to use that in combination with `networkmanager-iwd`.

Note that `networkmanager-iwd` is an AUR package that uses `iwd` exclusively
and does *not* require or depend on `wpa_supplicant` in any way.

### Configure iwd

First configure `iwd` defaults:

```shell
mkdir -p /etc/iwd
cat <<EOF > "/etc/iwd/main.conf"
[General]
RoamThreshold=-80
RoamThreshold5G=-80
CriticalRoamThreshold=-90
CriticalRoamThreshold5G=-90

[Network]
EnableIPv6=true
NameResolvingService=systemd

[Scan]
DisablePeriodicScan=true
DisableRoamingScan=true
MaximumPeriodicScanInterval=3600

[DriverQuirks]
PowerSaveDisable=iwlwifi
EOF
```

`iwd` can be configured using `iwctl`, or it is managed completely by
`networkmanager-iwd`. See the [iwd](https://wiki.archlinux.org/title/Iwd) page
on the Arch wiki for more details about `iwd`.

### Using networkmanager-iwd (optional)

I've opted to use `networkmanager-iwd` from AUR to manage my networking needs
because as it stands, it integrates much nicelier with graphical environments
like Gnome, Cosmic and KDE. To use `networkmanager-iwd`, you first need to
build the `networkmanager-iwd` package. See the section about how to
[configure for package building](#configuring-for-package-building) for more
information about how do that.

```shell
systemctl disable --now systemd-networkd # systemd-networkd needs to be off
systemctl disable --now iwd # iwd is managed by networkmanager itself
systemctl enable --now NetworkManager # enable networkmanager
```

See the [NetworkManager](https://wiki.archlinux.org/title/NetworkManager) page
on the Arch Wiki for details about how to use and configure NetworkManager. In
practice, you'll find it's pretty straight-forward and that it can usually be
configured graphically through your desktop environment of choice.

### Using systemd-networkd (optional)

As an alternative to `networkmanager-iwd`, you can use `systemd-networkd` to
configure your network. This comes down to placing configuration files in
`/etc/systemd/network`. and enabling the `systemd-networkd` services.

#### Setting up a single ethernet device with DHCP

```shell
cat <<EOF > "/etc/systemd/network/10-ethernet.network"
[Match]
Name=eth0

[Link]
RequiredForOnline=routable

[Network]
DHCP=yes
IgnoreCarrierLoss=10s
UseDomains=false

[DHCPv4]
RouteMetric=700
UseNTP=no
UseDNS=no
UseGateway=yes
UseRoutes=no

[IPv6AcceptRA]
RouteMetric=700
EOF
```

#### Setting up a bridge device with DHCP

Note that this is an alternative to the single ethernet configuration shown
previously. Let's first create the physical device bind config:

```shell
cat <<EOF > "/etc/systemd/network/10-bind.network"
[Match]
Name=eth0 eth1

[Network]
Bridge=bridge0

EOF
```

Create the virtual bridge device:

```shell
cat <<EOF > "/etc/systemd/network/10-bridge.netdev"
[NetDev]
Name=bridge0
Kind=bridge
MACAddress=52:41:41:46:00:0b
EOF
```

And finally, the bridge network, configured by DHCP:

```shell
cat <<EOF > "/etc/systemd/network/10-bridge.network"
[Match]
Name=bridge0

[Link]
RequiredForOnline=routable

[Network]
DHCP=yes
IgnoreCarrierLoss=10s
UseDomains=false

[DHCPv4]
RouteMetric=700
UseNTP=no
UseDNS=no
UseGateway=yes
UseRoutes=no

[IPv6AcceptRA]
RouteMetric=700
EOF
```

#### Setting up a wireless device with DHCP

Note that you first need to configure `iwd` to authenticate and connect to your
wireless network. After that, tell `systemd-networkd` about it:

```shell
cat <<EOF > "/etc/systemd/network/10-wireless.network"
[Match]
Name=wlan0

[Link]
RequiredForOnline=routable

[Network]
DHCP=yes
IgnoreCarrierLoss=10s
UseDomains=true

[DHCPv4]
RouteMetric=600

[IPv6AcceptRA]
RouteMetric=600
EOF
```

#### Enable systemd-networkd and iwd services

Enable `systemd-networkd` and `iwd `with:

```shell
systemctl disable --now NetworkManager # networkmanager needs to be off
systemctl enable --now iwd.service # iwd is not managed by systemd-networkd
systemctl enable --now systemd-networkd.service # enable systemd-networkd
systemctl disable --now systemd-networkd-wait-online.service # not this one
```

Note that we disable the waiting service, since we want to continue booting
even if there is no network. If you would like to know more about how you can
configure `systemd-networkd`, be sure to read the
[systemd-networkd](https://wiki.archlinux.org/title/Systemd-networkd) page on
the Arch wiki.

## Service configuration

I use and customize a bunch of services on my device. So I don't forget what
I customized and why, let me document it.

### DisplayLink

The `displaylink` service is used, together with the `evdi` driver, to handle
externally connected displays, which connect through a dock or other type of
USB3 or Thunderbolt connection. Enable it as follows:

```shell
systemctl enable --now displaylink
```

Additionally, I've observed that the `displaylink` service consumes an
inordinate amount of CPU time after a suspend/resume cycle. A quick restart of
the service works around that. To do that automatically, we can create a systemd
unit file and enable it:

```shell
# Write the systemd unit file for restarting displaylink on resume:
cat <<EOF > "/etc/systemd/system/displaylink-restart.service"
[Unit]
Description=Restart DisplayLink after resume
After=suspend.target

[Service]
Type=simple
ExecStart=/bin/systemctl --no-block restart displaylink.service

[Install]
WantedBy=suspend.target
EOF

# Enable and start the automatic displaylink restart service:
systemctl enable --now displaylink-restart
```

### CpuPower

The `cpupower` service reads from `/etc/default/cpupower` and configures the
default scheduler. Edit that file to set the default scheduler to `powersave` or
`performance `and then enable it:

```shell
systemctl enable --now cpupower.service
```

### Avahi

I don't want any service auto-configuring stuff on my system, especially things
like printers for example. Therefore I disable the Avahi zeroconf service:

```shell
systemctl mask avahi-daemon.service
systemctl mask avahi-daemon.socket
systemctl mask avahi-dnsconfd.service
```

Note that a side-effect of this, is that when you use something like `xsane` or
`simplescan`, you will get a password prompt dialog every time you start the
scanning tool. This is because, apparently, these tools use `avahi` somehow,
which then has to be invoked, which is then done through `polkit`, which raises
the password dialog so the `avahi` service can be started. Yeah.

### Bluetooth

Enable `bluetooth` with:

```shell
systemctl enable --now bluetooth.service
```

Bluetooth mostly just works out of the box, except for my XBox Series X|S
Wireless Game Controller. To get this to run, I use the `xpadneo-dkms` AUR
package. Additionally, I need to configure a few settings in
`/etc/bluetooth/main.conf` (add or set these settings yourself, or use `patch`
to apply the settings to your `main.conf`):

```diff
--- main.conf.orig  2023-07-31 19:01:29.473651656 +0300
+++ main.conf 2023-08-01 23:39:23.294653784 +0300
@@ -49,7 +49,7 @@
 # Restricts all controllers to the specified transport. Default value
 # is "dual", i.e. both BR/EDR and LE enabled (when supported by the HW).
 # Possible values: "dual", "bredr", "le"
-#ControllerMode = dual
+ControllerMode = dual

 # Maximum number of controllers allowed to be exposed to the system.
 # Default=0 (unlimited)
@@ -100,7 +100,7 @@
 # Specify the policy to the JUST-WORKS repairing initiated by peer
 # Possible values: "never", "confirm", "always"
 # Defaults to "never"
-#JustWorksRepairing = never
+JustWorksRepairing = confirm

 # How long to keep temporary devices around
 # The value is in seconds. Default is 30.
@@ -212,9 +212,9 @@

 # LE default connection parameters.  These values are superceeded by any
 # specific values provided via the Load Connection Parameters interface
-#MinConnectionInterval=
-#MaxConnectionInterval=
-#ConnectionLatency=
+MinConnectionInterval=7
+MaxConnectionInterval=9
+ConnectionLatency=0
 #ConnectionSupervisionTimeout=
 #Autoconnecttimeout=

@@ -318,7 +318,7 @@
 # AutoEnable defines option to enable all controllers when they are found.
 # This includes adapters present on start as well as adapters that are plugged
 # in later on. Defaults to 'true'.
-#AutoEnable=true
+AutoEnable=false

 # Audio devices that were disconnected due to suspend will be reconnected on
 # resume. ResumeDelay determines the delay between when the controller
```

After changes, restart the service:

```shell
systemctl restart bluetooth.service
```

### GDM

Enable `gdm` with:

```shell
systemctl enable --now gdm.service
```

I prefer auto-login on some of my devices (no on laptop, yes on desktop). Add
the two lines or apply the below diff to `/etc/gdm/custom.conf` using `patch`
if you would like your user to allow GDM to automatically login (don't forget
to replace `$USER` with your user name):

```diff
--- custom.conf.orig  2023-07-31 19:08:35.307832755 +0300
+++ custom.conf 2023-07-31 19:08:25.741219958 +0300
@@ -1,6 +1,8 @@
 # GDM configuration storage

 [daemon]
+AutomaticLogin=$USER
+AutomaticLoginEnable=True
 # Uncomment the line below to force the login screen to use Xorg
 #WaylandEnable=false
```

Reboot for the above to take effect.

### Ollama

The `ollama` service makes it possible to easily run AI models locally using
`llama-cpp`. Just like docker, this runs as a service and you use the `ollama`
command line utility to interact with it.

```shell
systemctl enable --now ollama.service
```

Remember to install `ollama-cuda` or `ollama-rocm` if you are using NVIDIA or
AMD devices respectively (see earlier chapter about that).

After the service is enabled and running, you can run a model as follows (we're
using the `deepcoder` model here as an example):

```shell
ollama run deepcoder
```

This will download the model and start an interactive prompt.

### Docker

First make sure `containerd` (a dependency for `docker`) does not load plugins
which are not relevant for this setup (these plugins cause loading errors in a
default setup like we're building - this does not break, but it does cause a
lot of regular error messages in `journalctl`):

```shell
mkdir -p /etc/containerd
cat <<EOF > "/etc/containerd/config.toml"

version = 3
disabled_plugins = ["io.containerd.snapshotter.v1.blockfile", "io.containerd.snapshotter.v1.btrfs", "io.containerd.snapshotter.v1.devmapper", "io.containerd.snapshotter.v1.erofs", "io.containerd.snapshotter.v1.zfs", "io.containerd.differ.v1.erofs", "io.containerd.tracing.processor.v1.otlp", "io.containerd.internal.v1.tracing", "io.containerd.grpc.v1.cri"]
EOF
```

Now we configure Docker to use a specific IP range for the default network (the
`bip` setting) and a bunch of range-reservations for custom created networks
(the `default-address-pools` setting). Additionally, we use `libvirtd`'s
`dnsmasq` instance to resolve DNS (see next section):

```shell
mkdir -p /etc/docker
cat <<EOF > "/etc/docker/daemon.json"
{
  "bip":"10.10.12.1/24",
  "dns" : [ "10.10.12.1" ],
  "default-address-pools": [
    { "base": "10.10.13.0/24", "size": 24 },
    { "base": "10.10.14.0/24", "size": 24 },
    { "base": "10.10.15.0/24", "size": 24 },
    { "base": "10.10.16.0/24", "size": 24 },
    { "base": "10.10.17.0/24", "size": 24 },
    { "base": "10.10.18.0/24", "size": 24 },
    { "base": "10.10.19.0/24", "size": 24 }
  ]
}
EOF
```

Enable docker with:

```shell
systemctl enable --now docker.service
```

I use docker without any further adjustments.

### Libvirtd

Enable `libvirtd` with:

```shell
systemctl enable --now libvirtd.service
```

I run `libvirtd` mostly stock. I do set `unix_sock_group` to `libvirt` and add
myself to the `libvirt` group. I then set `unix_sock_ro_perms`,
`unix_sock_rw_perms` and `unix_sock_admin_perms` to `0770` (Meaning, the owner
and group can read, write and execute, everybody else can do nothing).

```
unix_sock_group = "libvirt"
unix_sock_ro_perms = "0770"
unix_sock_rw_perms = "0770"
unix_sock_admin_perms = "0770"
```

You need to change this for a whole lot of files under `/etc/libvirt`. Not doing
this causes problems when connecting with your user instead of root using the
`virsh` or `virt-manager` clients. don't forget to restart the service after
changes:

```shell
systemctl restart libvirtd.service
```

Furthermore, I configure the `virt0` interface of the `default` NAT-enabled
network to have a specific IP address (10.10.11.1) and range (note: needs to
be run after starting/restarting libvirtd, and after configuring Docker; if
you look closely, you'll notice that this network configuration makes sure
that the `dnsmasq` service generated and spawned by `libvirtd` additionally
binds to the `docker0` network interface, which is an added convenience that
enables Docker containers in the default network to use this `dnsmasq` instance
to resolve DNS, as opposed to having to manage a separate/manual `dnsmasq`
instance for Docker):

```shell
export UUID=$(uuidgen)
cat <<EOF > "/tmp/net-default.xml"
<network xmlns:dnsmasq="http://libvirt.org/schemas/network/dnsmasq/1.0">
  <name>default</name>
  <uuid>$UUID</uuid>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virt0' stp='off' delay='0'/>
  <ip address='10.10.11.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='10.10.11.2' end='10.10.11.254'/>
    </dhcp>
  </ip>
  <dnsmasq:options>
    <dnsmasq:option value="interface=docker0"/>
  </dnsmasq:options>
</network>
EOF

virsh net-destroy default
virsh net-undefine default
virsh net-define /tmp/net-default.xml
virsh net-start default
rm -qf /tmp/net-default.xml
```

### OpenSSL

I did run into an issue connecting with older VPN environments related
to OpenSSL 3.x disabling various legacy encapsulation and connection modes by
default. The error you would see in such a case is:

```
Jul 31 20:26:38 FRAME openvpn[58956]: OpenSSL: error:11800071:PKCS12 routines::mac verify failure
Jul 31 20:26:38 FRAME openvpn[58956]: OpenSSL: error:0308010C:digital envelope routines::unsupported
Jul 31 20:26:38 FRAME openvpn[58956]: Decoding PKCS12 failed. Probably wrong password or unsupported/legacy encryption
```

Furthermore, when you are behind corporate proxies, you might also have
difficulties passing through the corporate proxy without the settings
`UnsafeLegacyRenegotiation` and `UnsafeLegacyServerConnect` (which were allowed
by default on OpenSSL 1.x).

To work-around these issues, I place a custom  `/etc/ssl/openssl.cnf`:

```shell
cp -n "/etc/ssl/openssl.cnf" "/etc/ssl/openssl.cnf.orig"
cat <<EOF > "/etc/ssl/openssl.cnf"
HOME = .
openssl_conf = openssl_init

[openssl_init]
providers = provider_sect
ssl_conf = ssl_sect

[provider_sect]
default = default_sect
legacy = legacy_sect

[default_sect]
activate = 1

[legacy_sect]
activate = 1

[ssl_sect]
system_default = system_default_sect

[system_default_sect]
Options = UnsafeLegacyRenegotiation,UnsafeLegacyServerConnect
EOF
```

Note that I would only do the above if you need to interact with some old
legacy VPN stuff, or if you're behind moron-grade SSL-terminating proxies.
{: .notice--warning}

### NFSv4

Enable `nfsv4` with:

```shell
systemctl enable --now nfsv4-server.service
```

Quick note: the above start may fail if you updated the `linux` package but did
not reboot yet, you'll see an error about a dependency failure.
{: .notice--warning}

I use NFS only on internal interfaces, specifically the `virt0` interface of
the `default` network (remember that ip address 10.10.11.1?). This allows me
to work with shared storage on other operating systems that I fool around with
on Qemu/KVM (Note that you have much better options for modern Linux systems -
there you can use `enable shared memory` and a `virtiofs` device to essentially
loop-mount a memory block device which is a directory on the host).

Since we're only doing NFSv4, and we're not interested in user/group ID mapping,
let's stop and mask a couple of RPC services first:

```shell
systemctl stop rpcbind.service
systemctl mask rpcbind.service
systemctl stop nfs-blkmap
systemctl mask nfs-blkmap
systemctl stop nfs-idmapd
systemctl mask nfs-idmapd
systemctl stop nfs-mountd
systemctl mask nfs-mountd
```

To make NFSv4 only listen on a specific interface, and to disable version 3 of
the protocol explicitly, we patch `/etc/nfs.conf` (use `patch` or add the
`host=`, `vers3=` and `vers4=` elements by hand under `[nfsd]`:

```diff
--- nfs.conf.orig       2023-07-31 21:16:20.438028044 +0300
+++ nfs.conf    2023-08-03 09:01:05.586457300 +0300
@@ -67,13 +67,14 @@
 # debug=0
 # threads=8
-# host=
+host=10.10.11.1
 # port=0
 # grace-time=90
 # lease-time=90
 # udp=n
 # tcp=y
-# vers3=y
-# vers4=y
+vers3=n
+vers4=y
 # vers4.0=y
 # vers4.1=y
 # vers4.2=y
```

Create an exports for /home:

```shell
cp -n "/etc/exports.d/home.exports" "/etc/exports.d/home.exports.orig"
cat <<EOF > "/etc/exports.d/home.exports"
/home   10.10.11.0/24(rw,sync,crossmnt,no_subtree_check)
EOF
```

After the above changes, restart the service:

```shell
systemctl restart nfsv4-server.service
```

### Samba

Enable `smbd` and `nmbd` with:

```shell
systemctl enable --now nmb.service smb.service
```

I use Samba for the same reasons as I use NFS, that is to have shared storage
on various older virtual machines (like Windows NT 4.0). Let's create an
`smb.conf` file:

```shell
cat <<EOF > "/etc/samba/smb.conf"
[global]
   workgroup = WORKGROUP
   netbios name = $(hostname | tr 'a-z' 'A-Z' | cut -d. -f1)
   server string =
   server role = standalone server
   server min protocol = NT1
   ntlm auth = yes
   lanman auth = yes
   hosts allow = 10.10.11.
   log file = /var/log/samba/log.smbd
   max log size = 100
   interfaces = 10.10.11.1/24
   bind interfaces only = yes
   dns proxy = yes
   wins proxy = yes
   wins support = yes
   local master = yes
   domain master = yes
   preferred master = yes
   os level = 33

[homes]
   comment = Home Directories
   acl allow execute always = True
   browsable = yes
   writable = yes
   valid users = %U
   create mask = 0644
   directory mask = 0755
EOF
```

After creating or changing `smb.conf`, restart the services:

```shell
systemctl restart nmb.service smb.service
```

### TLP

Enable `tlp` with:

```shell
systemctl enable --now tlp.service
```

TLP is used to manage power-saving modes of various hardware. It is usually
configured to enable power-saving when not connected to AC, and to disable it
when connected to AC. It does that for all kinds of things, like Wi-Fi, USB,
PCIe, Bluetooth, the CPU scheduler, etc.

I've made a custom TLP configuration for my Framework Laptop 13 which you can
install as follows:

```shell
cat <<EOF > "/etc/tlp.d/01-custom.conf"
CPU_SCALING_GOVERNOR_ON_AC=powersave
CPU_SCALING_GOVERNOR_ON_BAT=powersave

CPU_BOOST_ON_AC=0
CPU_BOOST_ON_BAT=0

PCIE_ASPM_ON_BAT=powersupersave

PLATFORM_PROFILE_ON_AC=balanced
PLATFORM_PROFILE_ON_BAT=low-power

USB_ALLOWLIST=32ac:0002
USB_EXCLUDE_BTUSB=1
USB_EXCLUDE_PRINTER=0

WIFI_PWR_ON_AC=off
WIFI_PWR_ON_BAT=off

WOL_DISABLE=N
EOF
```

After the above, you need to restart the service:

```shell
systemctl restart tlp.service
```

### Cups

Enable `cups` with:

```shell
systemctl enable --now cups.socket
```

I also don't want cups via `cups-browsed` to be able to auto-add printers, so I
patch `/etc/cups/cups-browsed.conf` as follows:

```diff
--- cups-browsed.conf.default	2023-08-29 09:57:47.157250003 +0200
+++ cups-browsed.conf	2023-08-29 09:58:45.054108764 +0200
@@ -53,7 +53,7 @@
 # BrowseLocalProtocols.
 # Can use DNSSD and/or CUPS and/or LDAP, or 'none' for neither.

-# BrowseProtocols none
+BrowseProtocols none


 # Only browse remote printers (via DNS-SD or CUPS browsing) from
```

### CDemu

I  use `cdemu` and related `vhba` kernel module to emulate optical drives for
use with emulation stuff. If you'd like to run CDemu, make sure the required
modules are loaded at boot:

```shell
# Load modules at boot.
cat <<EOF > "/etc/modules-load.d/cdemu.conf"
sg
sr_mod
vhba
EOF

# Do it now too.
modprobe -a sg sr_mod vhba

```

### Enabling user services

I use the following user-level services (do as logged in user):

```shell
for u in syncthing.service wireplumber.service pipewire.socket pipewire-pulse.socket; do systemctl enable --now --user $u; done
```

## How I use AUR

All the hip young things are running `yay` these days, but I like to do it the
bare-hands way. That way I have more feeling with what's going on with AUR
packages.

### Setting up your own custom local repository

First, let's create a `custom` repository source for pacman. Note that I sign
my packages using my GnuPG key, so the following assumes that. If you don't
want to sign your own packages, you need to change the `SigLevel` setting in
`/etc/pacman.conf` for your `custom` repository and you need to tell `makepkg`
not to sign your built package.

Also note that my package build root location is specific to my needs;
feel free to change it to anything you like.

```shell
# Set package build root location.
export PKG_ROOT="$HOME/Packaging/Arch"

# Create a directory structure for Arch packaging.
mkdir -p "$PKG_ROOT"
cd "$PKG_ROOT"
install -d Build Repository 'Source Packages' Sources -o $USER

# Create a repository database.
cd "$PKG_ROOT/Repository"
repo-add -s custom.db.tar.gz

# Add entry to /etc/pacman.conf.
cat <<EOF >> "/etc/pacman.conf"
[custom]
SigLevel = Required DatabaseRequired TrustedOnly
Server = file://$PKG_ROOT/Repository
EOF

# Add your public key to pacman keychain and set trust (assuming key matches $USER).
gpg --export --armor $USER > your.key
pacman-key --add your.key
pacman-key --lsign-key $USER
rm -qf your.key


# Run update for db and files, you should see custom being referenced.
pacman -Syu
pacman -Fy

# Unset variables.
unset PKG_ROOT
```

### Configuring for package building

Now we configure `makepkg` defaults. Make sure you configure the PKG_ROOT,
GPG_PUBKEY and PACKAGER environment variables to your specific likings:

```shell
# Set package build root location, gpgpubkey-id and packager string.
export PKG_ROOT="$HOME/Packaging/Arch"
export GPG_PUBKEY='14B189C4E877C9CAEA7F99C7ED3BDDB83BDD2604'
export PACKAGER='Rubin Simons <me@rubin55.org>'

cat <<EOF > "$HOME/.makepkg.conf"
# ~/.makepkg.conf.
MAKEFLAGS="-j$(nproc)"
BUILDENV=(!distcc color !ccache check sign)
BUILDDIR="$PKG_ROOT/Build"
PKGDEST="$PKG_ROOT/Repository"
SRCDEST="$PKG_ROOT/Sources"
SRCPKGDEST="$PKG_ROOT/Source Packages"
GPGKEY="$GPG_PUBKEY"
PACKAGER="$PACKAGER"
EOF

# Unset variables.
unset PKG_ROOT GPG_PUBKEY PACKAGER
```

### Example interactions using AUR

Here are a few example interactions with package fetching, building, repository
updating and package removal to get you started:

```shell
# Set package build root location.
export PKG_ROOT="$HOME/Packaging/Arch"

# Get an AUR package.
cd "$PKG_ROOT/Build"
git clone https://aur.archlinux.org/aurutils.git

# Build an AUR package.
cd aurutils
makepkg -cCs
ls ../../Repository/aurutils*

# Show information about a built package.
cd "$PKG_ROOT/Repository"
pacman -Qpi aurutils-*-any.pkg.tar.zst

# Build a source package.
cd aurutils
makepkg -cCsS
ls ../../Source\ Packages

# Update local repository (adds new packages, removes older ones).
cd "$PKG_ROOT/Repository"
repo-add -s -n -R custom.db.tar.gz *.zst

# Remove a specific AUR package from local repository
cd "$PKG_ROOT/Repository"
repo-remove -s custom.db.tar.gz aurutils
```

### AUR packages I build and install

So now to fill that `$PKG_ROOT/Build` directory with packages from AUR so we
can build some stuff and put it in our own repository:

```shell
# Set package build root location.
export PKG_ROOT="$HOME/Packaging"

# Git clone them all.
cd "$PKG_ROOT/Build"
export GITREPOS="adwaita-qt-git aic94xx-firmware akku alsa-hdspeconf amitools ares-emu aseprite asm-lsp ast-firmware astrojs-language-server astrojs-ts-plugin audacious-gtk3 audacious-plugins-gtk3 aurutils battop-bin blastem-hg bluez-hcitool bottles brscan4 bun-bin bundletool cartridges-git cdesktopenv-git chez-scheme claude-code clojure-lsp-bin cmake-format cmake-language-server codechecker conan crush cubeb cypher-shell davinci-resolve debtap dev-proxy-bin diagnostic-languageserver displaylink djmount dockerfile-language-server dolphin-emu-git dosbox-x-git dotool drawio-desktop-bin dumpasn1 earthly eclipse-platform elixir-ls erlang_ls evcxr_jupyter evdi-dkms-git exercism figma-linux-bin flow-bin flutterup flycast-git fmt9 fsautocomplete-bin ghcup-hs-bin github-copilot github-desktop gnome-shell-extension-auto-accent-color gnome-shell-extension-dash-to-dock-git gnome-shell-extension-stealmyfocus-git gnome-shell-extension-tiling-shell-git godot-mono-git google-cloud-cli google-earth-pro groovy-language-server-git gtk2 helm-ls ibmcloud-cli icaclient icoextract imhex infer-bin jdk25-graalvm-bin jdk25-openj9-bin jdtls jetbrains-toolbox kotlin-lsp-bin kubelogin lagrange leanux lemminx lib32-gperftools lib32-gtk2 lib32-intel-gmmlib lib32-intel-media-driver lib32-libpng12 libbacktrace-git libchdr-git libpng12 librashader libretro-beetle-cygne-git libretro-beetle-lynx-git libretro-beetle-ngp-git libretro-beetle-pcfx-git libretro-beetle-saturn-git libretro-bluemsx-git libretro-dosbox-pure-git libretro-lrps2-git libretro-swanstation-git libretro-uae-git libspng-git libwireplumber-4.0-compat license-wtfpl logisim-evolution-bin m64py marksman-git marsdev-git mednaffe mei-amt-check-git metals mistral-vibe mkinitcpio-firmware mksh mongodb-bin mongosh-bin moonlight-qt-git ms-sys nand2tetris ncurses5-compat-libs neo4j-community-bin neovim-symlinks nestopia netcoredbg networkmanager-iwd nvidia-patch-git ocenaudio-bin omnisharp-roslyn-bin opencode openmsx openshift-client-bin openshift-developer-bin openshift-pipelines-bin openvpn-update-systemd-resolved pandoc-bin papirus-folders passmark-performancetest-bin patool pcsx2-git pcsx-redux-git pegasus-frontend-git perlnavigator plutosvg plutovg postman-bin powercap powershell-bin powershell-editor-services pragmatapro-fonts prek protonplus protontricks-git ps3-disc-dumper-bin pupdate-bin pvsneslib-git pwvucontrol python-agent-client-protocol python-asgi-lifespan python-async-timeout python-backoff python-dataclasses-json python-eval-type-backport python-fvs python-httpx-sse python-m3u8 python-machine68k python-mcp python-mistralai python-mpegdash python-opencensus python-opentelemetry python-opentype-feature-freezer python-orjson-git python-pathvalidate python-pluginbase python-pyqtdarktheme-fork python-pysdl2 python-python-ffmpeg python-setuptools-reproducible python-sse-starlette python-steamgriddb python-tidalapi python-tree-sitter python-tree-sitter-bash python-uuid7 python-uv-dynamic-versioning python-vdf qt5-gamepad qt5-webchannel qt5-webengine qt5-websockets rabtap rapidyaml rcu-bin regionset rpcs3-git ruby-backport ruby-e2mmap ruby-jaro_winkler ruby-jekyll-include-cache ruby-jekyll-redirect-from ruby-jekyll-sitemap ruby-just-the-docs ruby-reverse_markdown ruby-solargraph ruby-yard-activesupport-concern ruby-yard-solargraph rusty-psn-bin ryujinx sameboy sbom-tool-bin scala-dotty scheme-langserver-bin sedutil shellcheck-bin sigtop-git skyscraper-git slack-desktop snd-hdspe-dkms-git soapui softhsm-git stc-syncthing-git sunshine-git svelte-language-server-git swift-bin talanoa-bin tidal-dl-ng tidal-hifi-bin tla-toolbox townsemu-git ttf-b612 ungoogled-chromium-bin upd72020x-fw uxplay v4l-utils-git vacuum vala-language-server vasm visual-studio-code-bin vkbasalt-cli vlink vtsls wd719x-firmware winetricks-git xcursor-dmz xdg-terminal-exec-git xpadneo-dkms yaml-language-server-git y-cruncher zeal-git"
for p in $GITREPOS; do git clone https://aur.archlinux.org/$p.git; done

# And my own version of openshift-codeready-bin/crc.
git clone https://github.com/rubin55/openshift-codeready-bin.git

# Build all (I wouldn't do this, I would initially enter one-by-one and do
# git log ; makepkg -cCs manually). Will result in packages under $PKG_ROOT/Repository.
cd "$PKG_ROOT/Build"
for p in *; do cd $p; makepkg -cCs; if [[ $? -ne 0 && $? -ne 13 ]]; then echo "$p did not go well, please fix..."; break; fi; cd - > /dev/null; done

# Update custom repository.
cd "$PKG_ROOT/Repository"
repo-add -s -n -R custom.db.tar.gz *.zst

# Update pacman databases.
pacman -Syu

# List all packages pacman sees in custom repository.
pacman -Sl custom

# Install all not-installed packages from custom repository.
pacman -S --needed $(pacman -Sl custom | grep -v installed | awk '{print $2}')
```

### Personal AUR packages

I maintain these packages (all on AUR, except for `openshift-codeready-bin`):

  * [cdesktopenv-git](https://aur.archlinux.org/packages/cdesktopenv-git)
  * [cypher-shell](https://aur.archlinux.org/packages/cypher-shell)
  * [dev-proxy-bin](https://aur.archlinux.org/packages/dev-proxy-bin)
  * [fsautocomplete-bin](https://aur.archlinux.org/packages/fsautocomplete-bin)
  * [gnome-shell-extension-tiling-shell-git](https://aur.archlinux.org/packages/gnome-shell-extension-tiling-shell-git)
  * [infer-bin](https://aur.archlinux.org/packages/infer-bin)
  * [jdk25-openj9-bin](https://aur.archlinux.org/packages/jdk25-openj9-bin)
  * [leanux](https://aur.archlinux.org/packages/leanux)
  * [marksman-git](https://aur.archlinux.org/packages/marksman-git)
  * [marsdev-git](https://aur.archlinux.org/packages/marsdev-git)
  * [mistral-vibe](https://aur.archlinux.org/packages/mistral-vibe)
  * [openshift-codeready-bin](https://github.com/rubin55/openshift-codeready-bin)
  * [openshift-developer-bin](https://aur.archlinux.org/packages/openshift-developer-bin)
  * [openshift-pipelines-bin](https://aur.archlinux.org/packages/openshift-pipelines-bin)
  * [passmark-performancetest-bin](https://aur.archlinux.org/packages/passmark-performancetest-bin)
  * [pragmatapro-fonts](https://aur.archlinux.org/packages/pragmatapro-fonts)
  * [pupdate-bin](https://aur.archlinux.org/packages/pupdate-bin)
  * [pvsneslib-git](https://aur.archlinux.org/packages/pvsneslib-git)
  * [python-agent-client-protocol](https://aur.archlinux.org/packages/python-agent-client-protocol)
  * [rcu-bin](https://aur.archlinux.org/packages/rcu-bin)
  * [rusty-psn-bin](https://aur.archlinux.org/packages/rusty-psn-bin)
  * [ruby-solargraph](https://aur.archlinux.org/packages/ruby-solargraph)
  * [ruby-yard-solargraph](https://aur.archlinux.org/packages/ruby-yard-solargraph)
  * [ruby-yard-activesupport-concern](https://aur.archlinux.org/packages/ruby-yard-activesupport-concern)
  * [sbom-tool-bin](https://aur.archlinux.org/packages/sbom-tool-bin)
  * [scheme-langserver-bin](https://aur.archlinux.org/packages/scheme-langserver-bin)
  * [svelte-language-server-git](https://aur.archlinux.org/packages/svelte-language-server-git)
  * [tidal-dl-ng](https://aur.archlinux.org/packages/tidal-dl-ng)
  * [talanoa-bin](https://aur.archlinux.org/packages/talanoa-bin)
  * [yaml-language-server-git](https://aur.archlinux.org/packages/yaml-language-server-git)

These are a few open source softwares I want to create AUR packages for:

  * [apache-artemis](https://artemis.apache.org/)
  * [winexe](https://sourceforge.net/projects/winexe/)
  * [vbcc](http://www.compilers.de/vbcc.html)

And I also want to create AUR packages for these commercial packages: 

  * [ares-commander](https://www.graebert.com/us/cad-software/ares-commander/)
  * [tvpaint-professional](https://www.tvpaint.com/)
  
And these development kits / toolchains:

  * [gbdk-2020-git](https://github.com/gbdk-2020/gbdk-2020)
  * [devkitPro-git](https://github.com/devkitPro/buildscripts)
  * [kallistios-git](https://github.com/KallistiOS/KallistiOS)
  * [jo-engine](https://github.com/johannes-fetz/joengine)
  * [libdragon-git](https://github.com/DragonMinded/libdragon)
  * [m68k-amigaos-gcc](https://github.com/AmigaPorts/m68k-amigaos-gcc)
  * [ps2dev-git](https://github.com/ps2dev/ps2dev)
  * [ps3toolchain-git](https://github.com/ps3dev/ps3toolchain)
  * [psn00bsdk-git](https://github.com/Lameguy64/PSn00bSDK)
  * [saturn-sdk-gcc-sh2](https://github.com/willll/Saturn-SDK-GCC-SH2)

## Final thoughts

After you've done (most of) the above, a `reboot` is in order; the system should
come up cleanly, without errors or stalls.

I've been using this setup for the last two years and it has been consistently
great. I get good battery life, The fan almost never comes on, sleep and resume
work reliably, bluetooth works with mouse, gamepad and headphones. I've bought a
1TB USB SSD for one of the slots on which I installed Arch Linux; I then use the
internal NVME for `/home`.
