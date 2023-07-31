---
title: Running Arch Linux on the Framework Laptop 13
date: 2023-07-31T11:40:00+02:00
last_modified_at: 2023-07-31T17:44:00+02:00
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

* TOC
{:toc}

## Why Arch Linux

Prior to my Framework Laptop adventures, I've been planning to move to Arch 
Linux for a while. I'm a long-time Linux desktop user. I started out with those
Red Hat CD-ROMs you'd buy at your local bookshop (this was around '96), I
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

At a certain point I had about 70 applications in there which became a burden 
to keep up to date, so I set out to find a distribution that packages as much 
as possible of the software I use, and where packaging it yourself is as simple 
as possible.

I definitely prefer a rolling-release distro. I checked out Nix, Gentoo and Arch
Linux. Nix I found really appealing (hard declarative) but I hated the syntax 
(maybe Guix one day - I like Scheme). 

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

Blah

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
net.ifnames=0 module_blacklist=cros_ec_lpcs,hid_sensor_hub acpi_osi="!Windows 2020" tpm_tis.interrupts=0 nvme.noacpi=1 mem_sleep_default=s2idle
```

### Note about mkinitcpio vs. dracut

Quick note about `mkinitcpio` and `mkinitcpio-busybox`: These are used to build
the kernel initrd image. They are fine for most use-cases. Just note that if you 
need working LVM RAID mirrors, I couldn't get that to work with these packages, 
so on my desktop box, I opted to use `dracut`, which in my experience has much 
saner + robuster setup of LVM-based configurations.

Note that there are no hooks by default for rebuilding dracut-based initrd
images, so you'd need to do that manually after `linux` kernel package updates.
For example:

```shell
export KERNEL_VERSION=$(pacman -Q linux | awk '{print $2}')
cp /lib/modules/$KERNEL_VERSION/vmlinuz /boot/vmlinuz-linux
dracut --kver $KERNEL_VERSION --force /boot/initramfs-linux-fallback.img
dracut --hostonly --no-hostonly-cmdline --kver $KERNEL_VERSION --force /boot/initramfs-linux.img
```

## Packages I install

You can install the following packages right after your system comes up first 
boot, or you could do it during install with `pacstrap` (it doesn't really 
matter, but in any case, make sure `core`, `contrib` and `multilib` are enabled 
in `/etc/pacman.conf` first):

```shell
pacman -S --needed acpi acpi_call-dkms acpid adwaita-qt5 adwaita-qt6 alsa-utils ansible-language-server ant audacious autoconf automake aws-cli bash bash-language-server bc bind bison blender bookworm btop bubblewrap cabal-install calibre cameractrls cheese cifs-utils clang cmake corkscrew cpio cpupower cue cups curl dagger dash dcraw deno desmume devtools discord distrobox dive dmidecode dmraid dnsmasq docker docker-buildx docker-compose dos2unix dosfstools dotnet-sdk dracut efibootmgr elixir emacs eog erlang ethtool evince extra-cmake-modules fakeroot fd file file-roller firefox flex foomatic-db-engine foomatic-db-nonfree-ppds foomatic-db-ppds fop fractal fs-uae fs-uae-launcher fwupd fzf gamemode gcc gcolor3 gdb gdm ghc ghidra gimp git glab gnome-backgrounds gnome-browser-connector gnome-calculator gnome-characters gnome-console gnome-control-center gnome-session gnome-settings-daemon gnome-shell gnome-shell-extensions gnome-system-monitor gnome-tweaks gnupg go gopls gparted gradle groovy guile gvfs gvfs-gphoto2 gvfs-mtp gvfs-nfs gvfs-smb gvim harfbuzz-cairo haskell-language-server hdparm helm hplip htop i2c-tools ifuse inkscape iperf3 iptables-nft irssi jdk17-openjdk jdk8-openjdk jfsutils jq k9s kafka kooha kubectl libcroco libindicator-gtk3 libreoffice-still libretro-beetle-pce libretro-beetle-psx-hw libretro-core-info libretro-desmume libretro-dolphin libretro-duckstation libretro-flycast libretro-mame libretro-mgba libretro-mupen64plus-next libretro-nestopia libretro-pcsx2 libretro-picodrive libretro-ppsspp libretro-sameboy libretro-scummvm libretro-snes9x libretro-yabause libva-mesa-driver libva-utils libvirt libvisual libxcrypt-compat linux-headers lldb lm_sensors lm_sensors lshw lsof ltrace lua-language-server make mame mame-tools man-db man-pages mattermost maven mbedtls2 mednafen mesa-utils mesa-vdpau mgba-qt minikube mitmproxy mono mono-msbuild moreutils mplayer mpv mtools multipath-tools mupen64plus mutter nasm nautilus neofetch netbeans network-manager-applet networkmanager networkmanager-openvpn nfs-utils ninja nmap nodejs npm ntfs-3g nuget nvme-cli openbsd-netcat openldap openssh openvpn p7zip pacutils pandoc-cli patch pavucontrol pciutils perl perl-net-dbus perl-x11-protocol pinentry pipewire pipewire-alsa pipewire-jack pipewire-pulse pipewire-v4l2 plymouth postgresql powertop ppsspp pyright python python-kubernetes python-ldap python-opengl python-pip python-pycryptodomex python-pyopenssl python-setuptools python-websockets python-wheel qbittorrent qemu-full qgnomeplatform-qt5 qgnomeplatform-qt6 qmc2 qt5-declarative qt5-tools qt5-wayland qt5-webchannel qt5-webengine qt5ct qt6-multimedia-ffmpeg qt6-tools qt6-wayland qt6ct quodlibet rabbitmq racket retroarch retroarch-assets-ozone retroarch-assets-glui ripgrep rsync ruby ruby-rake-compiler rust s-tui samba sane sbt scons screen scummvm sdl2_mixer signal-desktop smartmontools smbclient snes9x speedtest-cli squashfs-tools stack steam step-ca step-cli stern strace sudo syncthing tar texlive-bin texlive-core the_silver_searcher thunderbird tlp tmux traceroute tracker3-miners tree ttf-joypixels typescript typescript-language-server unixodbc unzip usbutils util-linux v4l-utils valgrind vdpauinfo virt-manager vlc vulkan-tools wake wgetpaste wireless_tools wireplumber wireshark-cli wireshark-qt wmctrl wol xclip xdg-desktop-portal-gnome xdg-user-dirs-gtk xdg-utils xdotool xorg-font-util xorg-fonts-100dpi xorg-mkfontscale xorg-server xorg-server-devel xorg-xauth xorg-xdpyinfo xorg-xdriinfo xorg-xev xorg-xfontsel xorg-xhost xorg-xinit xorg-xinput xorg-xkill xorg-xprop xorg-xrandr xorg-xrdb xorg-xset xorg-xsetroot xorg-xvinfo xorg-xwayland xorg-xwininfo xsane xsane-gimp xterm yarn yasm yq yt-dlp yuzu zig zip zls zstd
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
systemctl enable touchegg
systemctl start touchegg
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
systemctl enable fprintd
systemctl start fprintd
```

Note that the latest Goodix MOC fingerprint readers which the Framework Laptop
13 uses, use a firmware that is not compatible with Linux. For more information,
see [here](https://community.frame.work/t/fingerprint-reader-failing-to-register-on-13th-gen/34153/60).
<br/><br/>
Apparently, when one installs the Windows drivers for these things, the Windows
driver actually **downgrades** the firmware to a version that works with Linux..
<br/><br/>
Installing Windows 10 in a virtual machine and passing through the Fingerprint 
USB device and then installing the Windows [drivers](https://knowledgebase.frame.work/en_us/framework-laptop-bios-and-driver-releases-13th-gen-intel-core-BkQBvKWr3),
downgrades the firwmware, after which the device is usable in Linux.
{: .notice--warning}

If your fingerprint reader is working, you can continue to follow the steps 
[here](https://wiki.archlinux.org/title/Fprint) to set it up for usage.

### Additionally install on devices with Intel graphics

My Framework Laptop 13 is intel-based, so I install these packages additionally:

```shell
pacman -S --needed intel-gpu-tools vulkan-intel intel-media-driver libvdpau-va-gl thermald
```

### Additionally install on devices with AMD graphics

If you have an AMD device instead, you might want these ones:

```shell
pacman -S --needed radeontop vulkan-radeon
```

### Additionally install on devices with NVidia graphics

If you have an NVidia device instead, you might want these ones:

```shell
pacman -S --needed nvidia-utils nvidia-dkms
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

Blah

### Bluetooth

Blah

### GDM

Blah

### Libvirt

Blah

### NetworkManager

Blah

### NFSv4

Blah

### Samba

Blah

### Thermald

Blah

### TLP

Blah

### Cups

Blah

### Docker

Blah

### Enabling and disabling services

I enable the following system-wide services to start at bootup (do as root):

```shell
for s in bluetooth.service gdm.service libvirtd.service NetworkManager.service nfsv4.service nmb.service smb.service thermald.service tlp.service cups.socket docker.socket; do systemctl enable $s; done
```

I use the following user-level services (do as logged in user):

```shell
for u in syncthing.service wireplumber.service pipewire.socket pipewire-pulse.socket; do systemctl enable --user $u; done
```

## How I use AUR

All the hip young things are running 'yay' these days, but I like to do it the
bare-hands way, that way I have more feeling with what's going on with AUR
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
SigLevel = Required DatabaseOptional TrustedOnly
Server = file://$PKG_ROOT/Packages
EOF

# Add your public key to pacman keychain and set trust (assuming key matches $USER).
gpg --export --armor $USER > your.key
pacman-key --add your.key
pacman-key --lsign-key $USER
rm -qf your.key


# Run update to test, you should see custom being referenced.
pacman -Syu

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
repo-add -n -R custom.db.tar.gz *.zst

# Remove a specific AUR package from local repository
cd "$PKG_ROOT/Packages"
repo-remove custom.db.tar.gz aurutils
```

### AUR packages I build and install

So now to fill that `$PKG_ROOT/Build` directory with packages from AUR so we 
can build some stuff and put it in our own repository:

```shell
# Set package build root location.
export PKG_ROOT="$HOME/Syncthing/Packaging/Arch"

# Git clone them all.
cd "$PKG_ROOT/Build"
for p in akku ares-emu attract-git aurutils authy azure-cli chez-scheme conan cubeb dolphin-emu-git dosbox-x duckstation-git earthly eclipse-java elixir elixir-ls erlang_ls exercism flycast godot-mono-bin google-cloud-cli groovy-language-server-git ibmcloud-cli icaclient imhex irccloud-bin jdk17-graalvm-bin jdk17-jetbrains-bin jdk17-openj9-bin jdtls jetbrains-toolbox krew kubelogin lagrange libretro-beetle-lynx-git libretro-beetle-pcfx-git libretro-bluemsx-git libretro-dosbox-pure-git libretro-fsuae-git libspng license-wtfpl m64py mathematica mednaffe metals moonlight-qt ms-sys ncurses5-compat-libs nestopia openmsx parsec-bin passmark-performancetest-bin pcsx2-git pegasus-frontend-git postman-bin powershell-bin protonmail-bridge-bin python-patch-ng python-pluginbase python-pysdl2 rabtap rcu-bin rebar3 remark-language-server roomeqwizard rpcs3-git ruby-backport ruby-e2mmap ruby-jaro_winkler ruby-reverse_markdown ruby-solargraph ryujinx-git sameboy scala-dotty skyscraper-git soapui sublime-text-4 sunshine tla-toolbox townsemu-git ums ungoogled-chromium-bin visual-studio-code-bin vi-vim-symlink vmware-horizon-client vmware-keymaps xpadneo-dkms zeal-git zlib-ng zoom; do git clone https://aur.archlinux.org/$p.git; done

# Build all (I wouldn't do this, I would initially enter one-by-one and do 
# git log ; makepkg -cCs manually). Will result in packages under $PKG_ROOT/Build.
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

### No AUR packages available (yet)

I recently created my first AUR package: [rcu-bin](https://aur.archlinux.org/packages/rcu-bin).

I intend to create a few more in the coming weeks:

  * [artemis](https://activemq.apache.org/components/artemis/download/)
  * [pragmata-pro](https://fsd.it/shop/fonts/pragmatapro/)
  * [leanux](https://github.com/jmspit/leanux)
  * [vbcc](http://www.compilers.de/vbcc.html)
  * [winexe](https://sourceforge.net/projects/winexe/)
  * [tvpaint](https://www.tvpaint.com/)
  * [ares-commander](https://www.graebert.com/us/cad-software/ares-commander/)
