---
title: Running Arch Linux on the Framework Laptop 13
date: 2023-07-31T11:40:00+02:00
last_modified_at: 2023-12-31T23:59:59+02:00
categories:
  - blog
tags:
  - computers
  - linux
---

This article sums up why and how I run Arch Linux on [my new Framework Laptop 13](/blog/why-i-am-getting-a-framework/), which I received on the 3rd of this month.

I've been busy getting up+running and getting to know the device. TL;DR; I'm
*extremely* impressed and very happy with this laptop - it is by far one of the
best devices I've owned in a long time. It is near-silent, It uses ~3watts at
idle, it's fast, it's sturdy and it looks good!

This is going to be quite a long article, so here's a Table of Contents:

* TOC
{:toc}

## Why Arch Linux

Prior to my Framework Laptop adventures, I've been planning to move to Arch
Linux for a while. I'm a long-time Linux desktop user. I started out with those
Red Hat CD-ROMs you'd buy at your local bookshop (this was around '96). I
fooled around with SCO UnixWare, had a whole period of SGI IRIX after that and
then some distro-hopping to Debian, Fedora, Arch Linux (in 2007), Gentoo and
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
opted to use `systemd-boot` as boot manager and `NetworkManager` as my network
configuration tooling.

### Additional kernel parameters I use

Whichever boot manager you use, you might want to set a few extra kernel
parameters. I found the following additionals handle a bunch of stuff nicely
on my Framework Laptop 13:

```
net.ifnames=0 libata.allow_tpm=1 module_blacklist=cros_ec_lpcs,hid_sensor_hub acpi_osi="!Windows 2020" tpm_tis.interrupts=0 nvme.noacpi=1 mem_sleep_default=s2idle split_lock_detect=off
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
pacman -S --needed acpi acpi_call-dkms acpid alsa-utils ansible-language-server ant ardour audacious autoconf automake aws-cli bash bash-language-server bc bind bison blender btop bubblewrap cabal-install calibre cameractrls carla cdemu-client cdemu-daemon cdrdao cdrtools cifs-utils clang clinfo clojure cmake corkscrew cpio cpupower cue cups curl dagger dash dcraw debugedit delve deno desmume devtools discord distrobox dive dmidecode dmraid dnsmasq docker docker-buildx docker-compose dool dos2unix dosfstools dotnet-sdk dvd+rw-tools editorconfig-core-c efibootmgr elixir emacs-wayland erlang ethtool evince exfatprogs extra-cmake-modules fakeroot fd file file-roller fio firefox flex foomatic-db-engine foomatic-db-nonfree-ppds foomatic-db-ppds fop fractal fs-uae fs-uae-launcher furnace fwupd fzf gamemode gcc gdb gdm ghc ghidra gimp git git-lfs glab gnome-backgrounds gnome-browser-connector gnome-calculator gnome-characters gnome-control-center gnome-disk-utility gnome-session gnome-settings-daemon gnome-shell gnome-shell-extensions gnome-system-monitor gnome-terminal gnome-themes-extra gnome-tweaks gnupg go gopls go-tools gparted gradle groovy guile gvfs gvfs-gphoto2 gvfs-mtp gvfs-nfs gvfs-smb gvim handbrake harfbuzz-cairo haskell-language-server hdparm helix helm hoogle hplip htop i2c-tools ifuse inkscape iperf3 iptables-nft irssi jfsutils jq k9s kafka kubectl leiningen libcroco libindicator-gtk3 libreoffice-still libretro-beetle-pce libretro-beetle-psx-hw libretro-core-info libretro-desmume libretro-dolphin libretro-flycast libretro-mame libretro-mgba libretro-mupen64plus-next libretro-nestopia libretro-pcsx2 libretro-picodrive libretro-ppsspp libretro-sameboy libretro-scummvm libretro-snes9x libretro-yabause libva-mesa-driver libva-utils libvirt libvisual libxcrypt-compat linux-headers lldb lm_sensors loupe lshw lsof lsscsi ltrace lua-language-server make mame mame-tools man-db man-pages marksman mattermost-desktop maven mbedtls2 mednafen mesa-utils mesa-vdpau meson mgba-qt minikube mitmproxy mono mono-msbuild moreutils mplayer mpv mtools multipath-tools mupdf-tools mupen64plus mutter nasm nautilus neofetch netbeans net-tools networkmanager network-manager-applet networkmanager-openvpn nfs-utils ninja nmap nodejs npm ntfs-3g nuget nvchecker nvme-cli nvtop openbsd-netcat opencl-clhpp opencl-headers openldap openssh openvpn p7zip pacutils pandoc-cli patch patchelf pavucontrol pciutils perl perl-net-dbus perl-x11-protocol pinentry piper pipewire pipewire-alsa pipewire-jack pipewire-pulse pipewire-v4l2 papirus-icon-theme pkgconf postgresql powertop ppsspp python python-jsbeautifier python-kubernetes python-ldap python-nose python-opengl python-pip python-pycryptodomex python-pylint python-pyopenssl python-pytest python-rope python-setuptools python-websockets python-wheel qbittorrent qemu-full qjackctl qmc2 qpwgraph qt5ct qt5-declarative qt5-tools qt5-wayland qt5-webchannel qt5-webengine qt6ct qt6-multimedia-ffmpeg qt6-tools qt6-wayland quodlibet rabbitmq racket realtime-privileges retroarch retroarch-assets-glui retroarch-assets-ozone ripgrep rlwrap rsync ruby ruby-rake-compiler rust rust-analyzer samba sane sbt scons screen scummvm sdl2_mixer seahorse shellcheck signal-desktop smartmontools smbclient snapshot snes9x speedtest-cli squashfs-tools stack steam step-ca step-cli stern strace s-tui stylelint sudo syncthing sysprof tar texlab texlive-bin texlive-core the_silver_searcher thunderbird tidy tmux traceroute tracker3-miners tree ttf-ibm-plex ttf-joypixels typescript typescript-language-server udftools uncrustify unixodbc unzip urlwatch usbutils util-linux v4l-utils valgrind vdpauinfo vhba-module-dkms virt-manager vscode-css-languageserver vscode-html-languageserver vscode-json-languageserver vulkan-tools wake wayland-utils wgetpaste whois wine wine-mono winetricks wireless_tools wireplumber wireshark-cli wireshark-qt wmctrl wol xchm xclip xdg-desktop-portal-gnome xdg-user-dirs-gtk xdg-utils xdotool xfsprogs xorg-fonts-100dpi xorg-font-util xorg-mkfontscale xorg-fonts-misc xorg-server xorg-server-devel xorg-xauth xorg-xdpyinfo xorg-xdriinfo xorg-xev xorg-xfontsel xorg-xhost xorg-xinit xorg-xinput xorg-xkill xorg-xprop xorg-xrandr xorg-xrdb xorg-xset xorg-xsetroot xorg-xvinfo xorg-xwayland xorg-xwininfo xsane xsane-gimp xterm yaml-language-server yarn yasm yq yt-dlp ython-lsp-server yuzu zig zip zls zstd
```

### Additionally install when you want dynamic application of power management settings

You can install `tlp` to enable dynamically applying power management settings,
based on if your power connector is connected or if you're on battery.

```shell
pacman -S --needed tlp
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
pacman -S --needed intel-gpu-tools vulkan-intel intel-media-driver libvdpau-va-gl intel-graphics-compiler intel-compute-runtime
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
pacman -S --needed radeontop vulkan-radeon
```

### Additionally install on devices with NVidia graphics

If you have an NVidia device instead, you might want these ones:

```shell
pacman -S --needed cuda cuda-tools ffnvcodec-headers libva-nvidia-driver nvidia-cg-toolkit nvidia-settings nvidia-utils nvidia-dkms nvtop opencl-nvidia
```

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

## Service configuration

I use and customize a bunch of services on my device. So I don't forget what
I customized and why, let me document it.

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
be run after starting/restarting libvirtd):

```shell
export UUID=$(uuidgen)
cat <<EOF > "/tmp/net-default.xml"
<network>
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
</network>
EOF

virsh net-destroy default
virsh net-undefine default
virsh net-define /tmp/net-default.xml
virsh net-start default
rm -qf /tmp/net-default.xml
```

### NetworkManager

Enable `NetworkManager` with:

```shell
systemctl enable --now NetworkManager.service
```

NetworkManager works mostly out of the box, normally no special settings needed.

However I did run into an issue connecting with older VPN environments related
to OpenSSL 3.x disabling various legacy encapsulation and connection modes by
default. The error you would see in such a case is:

```
Jul 31 20:26:38 FRAME nm-openvpn[58956]: OpenSSL: error:11800071:PKCS12 routines::mac verify failure
Jul 31 20:26:38 FRAME nm-openvpn[58956]: OpenSSL: error:0308010C:digital envelope routines::unsupported
Jul 31 20:26:38 FRAME nm-openvpn[58956]: Decoding PKCS12 failed. Probably wrong password or unsupported/legacy encryption
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

### Docker

Enable bluetooth with:

```shell
systemctl enable --now docker.socket
```

I use docker without any adjustments.

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
export PKG_ROOT="$HOME/Syncthing/Packaging/Arch"

# Create a directory structure for Arch packaging.
mkdir -p "$PKG_ROOT"
cd "$PKG_ROOT"
install -d Build Packages 'Source Packages' Sources -o $USER

# Create a repository database.
cd "$PKG_ROOT/Packages"
repo-add custom.db.tar.gz

# Add entry to /etc/pacman.conf.
cat <<EOF >> "/etc/pacman.conf"
[custom]
SigLevel = Required DatabaseRequired TrustedOnly
Server = file://$PKG_ROOT/Packages
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
export PKG_ROOT="$HOME/Syncthing/Packaging/Arch"
export GPG_PUBKEY='89E5EB2541BC6668A9C165D424BD51CD12534CE6'
export PACKAGER='Rubin Simons <me@rubin55.org>'

cat <<EOF > "$HOME/.makepkg.conf"
# ~/.makepkg.conf.
MAKEFLAGS="-j$(nproc)"
BUILDENV=(!distcc color !ccache check sign)
BUILDDIR="$PKG_ROOT/Build"
PKGDEST="$PKG_ROOT/Packages"
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
export PKG_ROOT="$HOME/Syncthing/Packaging/Arch"

# Get an AUR package.
cd "$PKG_ROOT/Build"
git clone https://aur.archlinux.org/aurutils.git

# Build an AUR package.
cd aurutils
makepkg -cCs
ls ../../Packages/aurutils*

# Show information about a built package.
cd "$PKG_ROOT/Packages"
pacman -Qpi aurutils-*-any.pkg.tar.zst

# Build a source package.
cd aurutils
makepkg -cCsS
ls ../../Source\ Packages

# Update local repository (adds new packages, removes older ones).
cd "$PKG_ROOT/Packages"
repo-add -n -R -s custom.db.tar.gz *.zst

# Remove a specific AUR package from local repository
cd "$PKG_ROOT/Packages"
repo-remove -s custom.db.tar.gz aurutils
```

### AUR packages I build and install

So now to fill that `$PKG_ROOT/Build` directory with packages from AUR so we
can build some stuff and put it in our own repository:

```shell
# Set package build root location.
export PKG_ROOT="$HOME/Syncthing/Packaging/Arch"

# Git clone them all.
cd "$PKG_ROOT/Build"
for p in adwaita-qt-git akku ares-emu attract-git aurutils authy azure-cli binder_linux-dkms bluez-hcitool chez-scheme clojure-lsp-bin cmake-format cmake-language-server conan cubeb djmount dockerfile-language-server dolphin-emu-git dosbox-x duckstation-git earthly eclipse-java edid-decode-git elixir-ls erlang_ls exercism flutter flycast gnome-shell-performance godot-mono-bin google-cloud-cli groovy-language-server-git hax11-git ibmcloud-cli icaclient imhex irccloud-bin jdk17-graalvm-bin jdk17-jetbrains-bin jdk17-openj9-bin jdk21-graalvm-bin jdk21-jetbrains-bin jdk21-openj9-bin jdtls jetbrains-toolbox krew kubelogin lagrange lemminx lib32-gperftools lib32-intel-gmmlib lib32-intel-media-driver libgbinder libglibutil libretro-beetle-lynx-git libretro-beetle-pcfx-git libretro-bluemsx-git libretro-dosbox-pure-git libretro-fsuae-git libspng-git license-wtfpl m64py mathematica mednaffe mei-amt-check-git metals moonlight-qt ms-sys mutter-performance ncurses5-compat-libs nestopia netcoredbg nvidia-utils-nvlax omnisharp-roslyn-bin openmsx openshift-client-bin openshift-codeready-bin openshift-developer-bin openshift-pipelines-bin papirus-folders parsec-bin passmark-performancetest-bin pcsx2-git pegasus-frontend-git perlnavigator postman-bin powercap powershell-bin pragmatapro-fonts protonmail-bridge-bin protonplus protontricks ps3-disc-dumper-bin python-gbinder python-grip python-hwdata python-patch-ng python-path-and-address python-pluginbase python-pysdl2 python-vdf rabtap rcu-bin rebar3 regionset remark-language-server roomeqwizard rpcs3-git ruby-backport ruby-e2mmap ruby-jaro_winkler ruby-rbs ruby-reverse_markdown ruby-solargraph ryujinx-git sameboy scala-dotty scala-scala3-symlink scheme-chez-symlink sedutil skyscraper-git soapui sublime-text-4 sunshine tla-toolbox townsemu-git ttf-b612 tuxclocker ums ungoogled-chromium-bin vala-language-server visual-studio-code-bin vi-vim-symlink vmware-horizon-client vmware-keymaps waydroid xcursor-dmz xpadneo-dkms y-cruncher zeal-git zlib-ng zoom; do git clone https://aur.archlinux.org/$p.git; done

# Build all (I wouldn't do this, I would initially enter one-by-one and do
# git log ; makepkg -cCs manually). Will result in packages under $PKG_ROOT/Packages.
cd "$PKG_ROOT/Build"
for p in *; do cd $p; makepkg -cCs; cd -; done

# Update custom repository.
cd "$PKG_ROOT/Packages"
repo-add -n -R custom.db.tar.gz *.zst

# Update pacman databases.
pacman -Syu

# List all packages pacman sees in custom repository.
pacman -Sl custom

# Install all not-installed packages from custom repository.
pacman -S --needed $(pacman -Sl custom | grep -v installed | awk '{print $2}')
```

### Personal AUR packages

I maintain these AUR packages:

  * [openshift-codeready-bin](https://aur.archlinux.org/packages/openshift-codeready-bin)
  * [openshift-developer-bin](https://aur.archlinux.org/packages/openshift-developer-bin)
  * [openshift-pipelines-bin](https://aur.archlinux.org/packages/openshift-pipelines-bin)
  * [pragmata-pro](https://aur.archlinux.org/packages/pragmatapro-fonts)
  * [rcu-bin](https://aur.archlinux.org/packages/rcu-bin)
  * [scala-scala3-symlink](https://aur.archlinux.org/packages/scala-scala3-symlink)
  * [scheme-chez-symlink](https://aur.archlinux.org/packages/scheme-chez-symlink)

Here are a few more things I plan to create AUR packages for:

  * [artemis](https://activemq.apache.org/components/artemis/download/)
  * [leanux](https://github.com/jmspit/leanux)
  * [vbcc](http://www.compilers.de/vbcc.html)
  * [winexe](https://sourceforge.net/projects/winexe/)
  * [tvpaint](https://www.tvpaint.com/)
  * [ares-commander](https://www.graebert.com/us/cad-software/ares-commander/)

## Final thoughts

After you've done (most of) the above, a `reboot` is in order; the system should
come up cleanly, without errors or stalls.

I've be been using this setup for the last month and it has been pretty great.
I get really good battery life, The fan almost never comes on, sleep and resume
work reliably, bluetooth works with mouse, gamepad and headphones, Libvirt has
been amazing with a bunch of interesting virtual machines (Guix, Windows NT,
RHEL6 with Softimage).

I hope to be using this machine and operating system for a long time!
