# Raspberry Pi

![](https://camo.githubusercontent.com/9e96cf15fbd387092abc4fd6ede84e29cee79892d4d912d0a44b63ca7fe8cb5e/68747470733a2f2f7062732e7477696d672e636f6d2f6d656469612f457a48427433575845414d7131556e3f666f726d61743d6a7067266e616d653d736d616c6c)

Although we are currently not providing installation images for Raspberry Pi, we are investigating whether doing so would be feasible and beneficial.

The same instructions are roughly applicable to other aarch64 based systems such as rk3399 ones like the PineBook Pro and the [pine64 rockpro64_rk3399](https://bsd-hardware.info/?id=board:pine64-pinebook-pro-rk3399) as well.

> __Note:__ This page is intended for technically advanced users and developers, not for end users. This page is work in progress. Instructions are brief and assume prior knowledge with the subject.

This page describes how to run the main components of helloSystem (referred to collectively as helloDesktop) on Raspberry Pi 3 and Raspberry Pi 4 devices, starting with the official FreeBSD 13 image that does not even contain Xorg.

It describes how to build and set up the main components of helloSystem by hand, so this is a good walkthrough in case you are interested in bringing up the helloDesktop on other systems as well.

If there is enough interest, we might eventually provide readymade helloSystem images for Raspberry Pi in the future. This depends on how usable helloSystem will be on the Raspberry Pi (e.g., whether we can get decent graphics performance).

## Prerequisites

* Raspberry Pi 3 Model B or B+, Raspberry Pi 4 Model B, or Raspberry Pi 400
* Suitable power supply
* Class-1 or faster microSD card with at least 8 GB of memory
* HDMI display
* Keyboard and mouse (we recommend the official Raspberry Pi keyboard since helloSystem can automatically detect its language)
* Ethernet (WLAN is not yet covered on this page)

## Optional hardware

* SATA SSD drive (not NVME)

## Preparing an SD card

* Install the official FreeBSD 13 image (not FreeBSD 14) to a microSD card
* Copy this file to the microSD card you are running the Raspberry Pi from. You will need it because you will not have an internet browser at hand
* Edit `config.txt` in the `MSDOSBOOT` partition to contain the correct resolution for your screen.

```
hdmi_safe=0
disable_overscan=1
display_auto_detect=1
max_framebuffer_width=3840
max_framebuffer_height=2160
max_framebuffers=2
gpu_mem=512

[pi4]
dtoverlay=vc4-kms-v3d-pi4
```

``` .. note::
If, for some reason, the display does not work, simply remove or comment out the `max_framebuffer*` lines and replace them with the following:
```
```
framebuffer_width=<display resolution width>
framebuffer_height=<display resolution height>
```

## Preparing a bootable SSD drive (optional but recommended)

* [Update your firmware if USB boot is not working](https://www.raspberrystreet.com/learn/how-to-boot-raspberrypi-from-usb-ssd)
* Follow the instructions in the SD card section, but write the image to your SSD

``` .. note::
The Raspberry Pi 4/400 doesn't have a PCIe bus, so booting off an NVMe drive over USB protocol is not supported by the hardware. You will need to use a SATA SSD. This works for either M.2 or traditional drive form factors.
```

## Booting into the system

We will be doing all subsequent steps on the Raspberry Pi system itself since we are assuming you don't have any other 64-bit ARM (`aarch64`) machines at hand.

* Insert the microSD card into the Raspberry Pi
* Attach keyboard and mouse (we recommend the official Raspberry Pi keyboard since helloSystem can automatically detect its language)
* Attach the display (If you don't see anything, DO NOT power off your machine. Simply switch the HDMI port.)
* Attach Ethernet
* Power on the Raspberry Pi
* After some time, you should land in a login screen
* Log in with `root`, password `root`
* Install some basic software with `pkg install -y xorg xterm nano openbox fluxbox git-lite`

``` .. note::
`fluxbox` is not strictly needed and can be removed later on, but it gives us a graphical desktop session while we are working on installing helloDesktop
```

## Starting a graphical session

Start Xorg with

```
echo startfluxbox > ~/.xinitrc
startx
```

Doing it like this is the easiest way since the display `DISPLAY` environment variable and other things will be set automatically.

## Compiling and installing helloDesktop core components

### Install prerequisites
    
```
pkg install -y cmake pkgconf qt5-qmake qt5-buildtools kf5-kdbusaddons kf5-kwindowsystem libdbusmenu-qt5 qt5-concurrent qt5-quickcontrols2 libfm libqtxdg wget
```
    
### launch

The `launch` command is central to helloSystem and is used to launch applications throughout the system.

```
git clone https://github.com/helloSystem/launch
cd launch/
mkdir build
cd build/
cmake ..
make -j4
./launch
make install
cd ../../
```

### Menu

```
git clone https://github.com/helloSystem/Menu
cd Menu
mkdir build
cd build
cmake ..
make -j4
make install
ln -s /usr/local/bin/menubar /usr/local/bin/Menu # Workaround for the 'launch' command to find it
cd ../../
```

### Filer

```
git clone https://github.com/helloSystem/Filer
cd Filer/
mkdir build
cd build/
cmake ..
make -j4
make install
ln -s /usr/local/bin/filer-qt /usr/local/bin/Filer # Workaround for the 'launch' command to find it
cd ../../
```

``` .. note::
It seems like Filer refuses to start if D-Bus is missing. We should change it so that it can also work without D-Bus and at prints a clear warning if D-Bus is missing. Currently all you see if D-Bus is missing is the following: '** (process:3691): WARNING **:  The directory '~/Templates' doesn't exist, ignoring it', and then Filer exits.
```
 
### Dock

```
pkg install -y <...>
git clone https://github.com/helloSystem/Dock
cd Dock
mkdir build
cd build
cmake ..
make -j4
make install
# Currently Dock is installed to /usr/bin/cyber-dock. In the future is should be in /usr/local/bin/.
ln -s /usr/bin/cyber-dock /usr/local/bin/Dock # Workaround for the 'launch' command to find it
cd ../../
```

### QtPlugin

```
git clone https://github.com/helloSystem/QtPlugin
cd QtPlugin/
mkdir build
cd build/
cmake ..
make -j4
make install
cp ../stylesheet.qss /usr/local/etc/xdg/
cd ../../
```

### Icons and other assets

Install icons and other helloSystem assets that are not packaged as FreeBSD packages (yet). It is easiest to run `bash`, then `export uzip=/`, and then run the corresponding sections of [script.hello](https://raw.githubusercontent.com/helloSystem/ISO/experimental/settings/script.hello). Here are some examples.

For the system icons:

```
wget -c -q http://archive.ubuntu.com/ubuntu/pool/universe/x/xubuntu-artwork/xubuntu-icon-theme_16.04.2_all.deb
tar xf xubuntu-icon-theme_16.04.2_all.deb
tar xf data.tar.xz
mkdir -p "${uzip}"/usr/local/share/icons/
mv ./usr/share/icons/elementary-xfce "${uzip}"/usr/local/share/icons/
mv ./usr/share/doc/xubuntu-icon-theme/copyright "${uzip}"/usr/local/share/icons/elementary-xfce/
sed -i -e 's|usr/share|usr/local/share|g' "${uzip}"/usr/local/share/icons/elementary-xfce/copyright
rm "${uzip}"/usr/local/share/icons/elementary-xfce/copyright-e

wget "https://raw.githubusercontent.com/helloSystem/hello/master/branding/computer-hello.png" -O "${uzip}"/usr/local/share/icons/elementary-xfce/devices/128/computer-hello.png
```

For the system fonts:

```
wget -c -q https://github.com/ArtifexSoftware/urw-base35-fonts/archive/20200910.zip
unzip -q 20200910.zip
mkdir -p "${uzip}/usr/local/share/fonts/TTF/"
cp -R urw-base35-fonts-20200910/fonts/*.ttf "${uzip}/usr/local/share/fonts/TTF/"
rm -rf urw-base35-fonts-20200910/ 20200910.zip
```

Proceed similarly for the cursor theme, wallpaper, applications from the helloSystem/Utilities repository, and other assets.

### Installing required packages

Install every package that is not commented out in [packages.hello](https://raw.githubusercontent.com/helloSystem/ISO/experimental/settings/packages.hello) that are installable.

``` .. note::
    Not every package will be installable. This probably means that the package in question has not been compiled for the `aarch64` architecture on FreeBSD 13 yet.
```

You can use this script to automatically install all listed packages:

```sh
url="https://raw.githubusercontent.com/helloSystem/ISO/experimental/settings/packages.hello"
curl -fsS $url | egrep -v '^#' | while read line ; do
  pkg install -y -U $line || echo "[error] $line cannot be installed"
done
```

To enable `automount`, run `service devd restart`.

### Installing overlays

```
git clone https://github.com/helloSystem/ISO
cp -Rfv ISO/overlays/uzip/hello/files/ / 
cp -Rfv ISO/overlays/uzip/localize/files/ / 
cp -Rfv ISO/overlays/uzip/openbox-theme/files/ /
cp -Rfv ISO/overlays/uzip/mountarchive/files/ / 
```

``` .. note::
    If we provided the temporary packages that are used in the helloSystem ISO build process for download, then one could just install those instead of having to do the above.
```

## Adjusting fstab

Edit `/etc/fstab` to look like this:

```
/dev/mmcsd0s2a  /       ufs     rw      0       0
```

Otherwise the root device may not be found upon reboot.

### Enabling D-Bus and Avahi

Filer does not launch and Zeroconf does not work if `/usr/local/bin/dbus-daemon --system` is not running. Hence add the following lines to `/etc/rc.conf' to enable it:

```
dbus_enable="YES"
avahi_enable="YES" 
```

and then start it `service dbus start`.

### Editing start-hello

Edit the `start-hello` script to use the `launch` command instead of hardcoding `/Applications/...`.

```
#################################
# Details to be inserted here.
# This script is work in progress.
#################################
```

### Starting helloDesktop session

```
echo start-hello > ~/.xinitrc
killall Xorg
```

Type `startx` to start an Xorg session; it should now load a helloDesktop session.

Sometimes it does not load all the way, in this case it is necessary to start xterm (right-click the desktop) and invoke `start-hello` from there again. The reason for this is unknown.

### Adding a user

Added a new user with Applications -> Preferences -> Users

```
cp /home/user/.openbox* .
cp -r /home/user/.config/* .config/
```

## Improving system performance

### Setting up SWAP for greater system memory capabilities

``` .. note::
    While this will work on an SD card, it will be slow and wear out your drive quickly. Take it from me, SD cards SUCK for quick memory page access. It is highly recommended to use a SATA SSD when using SWAP. This section was adapted from [a guide on nixCraft](https://www.cyberciti.biz/faq/create-a-freebsd-swap-file/). It goes into more detail and even breifly covers encryption.
```

* Create a SWAP file using
```
# dd if=/dev/zero of=/swap bs=1M count=8192
```
``` .. note::
This will create a swap file holding up to 8192 MB (8 GB) of memory. To go much further beyond that would recommend some further tweaking. 8 GB should be more than enough. It's easy enough to convert what you want in GB to MB, but for the sake of convenience, multiply how much memory you want in gigabytes (GB) by 1024. For example, if you wanted 4 GB, you'd do 4*1024=4096.
```
* Run `# chmod 0600 /swap` to give it the correct permissions.
* Edit /etc/fstab and append the following line:
```
md42	none	swap	sw,file=/swap	0	0
```
* Run `# swapon -aq` to immediately enable SWAP
* Run `$ swapinfo -k` to verify that it's been attached correctly

### Overclocking your system

With an SSD, SWAP, and several GPU enhancements enabled, you should immediately notice a speed boost. This will give you a much better development system and may even replace your desktop. However, you can quickly and easily take it a step further by overclocking your system. Please note, overclocking can lead to reduced hardware lifespan and can cause permenant damage. That said, I've found that a decent heat sing and a good fan go the trick. This is necessary as not having a somewhat competatant cooling solution will bite you in the ass. If you let it heat too much, it will thermal throttle and kill your performance. Do keep in mind, you need a good power supply to run this thing, especially overclocked. A Raspberry Pi power supply is a little different than a normal phone charger by design. Your new iPhone or MacBook charger probably won't cut it. I'd recommend getting the [official Rasbperry Pi power supply](https://www.amazon.com/Raspberry-Power-Supply-USB-C-Listed/dp/B07Z8P61DQ), manufactured by the Rasbperry Pi Foundation or whatever. Ir's OEM, it's good. Another great option is the [Argon ONE](https://www.amazon.com/Argon-Raspberry-Supply-Listed-Adapter/dp/B07TW4Q693) power supply.

Overclocking is inherently dangerous as you are pushing the device beyond the intended spec. However, it is a very simple and robust machine that will most likely experience no ussues doing this. The Raspberry Pi is very affordable and modular, making the risk of hardware failure less of an issue. The main board can easily be swapped out and EEPROM flashed and you have your whole system back. There's a hard limit to how far you can push your Pi, so don't even worry about overdoing it. No 8 GHz liquid nitrogen overclocks I'm afraid.

To overclock you Raspberry Pi 4 to the max, edit `/boot/msdos/config.txt` and append the following:

```
over_voltage=6
arm_freq=2147
gpu_freq=700
```

2147 MHz is the maximum the device can go. Unlike Raspberry Pi OS, FreeBSD will boot even if you set these values higher. Even though you can set it higher, I can see no difference, even at 3000. You can set the gpu_freq up to "900" before it refuses to boot on FreeBSD. However, I can't tell if it is actually working or not. Even if you can push it past its limits, you will probaby see diminishing returns. In the event that you actually can overclock it to these badass levels, I'd recommend investing in a monster CPU cooler, maybe even running liquid cooling or higher as 2147 MHz is already pretty warm.

## Known issues

``` .. note::
    Any help in improving the situation is appreciated.
```

Not sure whether `vc4-kms-v3d` is usable with FreeBSD.

Also see: https://papers.freebsd.org/2019/bsdcan/vadot-adventure_in_drmland/

### Missing login window

The `slim` package is missing, hence we don't have a login window at the moment.

``` .. note::
    Web browser performance may be improved and the following issues may not be relevant anymore. This will require more user feedback and testing.
```

### Web browser

The Falkon browser can be launched, but trying to load more than the most basic web pages such as http://frogfind.com/ leads to an instant crash:

```
# cat /root/.config/falkon/crashlog/Crash-2021-04-17T10:21:20.txt
Time: Sa. Apr. 17 10:21:20 2021
Qt version: 5.15.2 (compiled with 5.15.2)
Falkon version: 3.1.0
Rendering engine: QtWebEngine

============== BACKTRACE ==============
#0: 0x213810 <???> at /usr/local/bin/falkon
#1: 0x499b4994 <_pthread_sigmask+0x524> at /lib/libthr.so.3
```

``` .. note::
    The exact same crash also happens on a RockPro64 with 4GB of RAM.
```

### RAM compression

On Linux, there is also __zram__ which is enabled by default on Android and Chromium OS. Did anything ever come out of https://wiki.freebsd.org/SummerOfCode2019Projects/VirtualMemoryCompression?

macOS also does RAM compression when memory is getting low: https://www.lifewire.com/understanding-compressed-memory-os-x-2260327.

Windows also does RAM compression: https://www.tenforums.com/windows-10-news/17993-windows-10-memory-compression.html.

The following seems to "work" but can make the Raspberry Pi unacceptably slow:

```
mdconfig -a -t malloc -o compress -o reserve -s 700m -u 7
swapon /dev/md7
```

``` .. note::
    Please let us know if you know how to properly use RAM compression on FreeBSD.
```

``` .. note::
    Do not use ZFS on memory-constrained machines.
```

### Missing icons

Set `QT_QPA_PLATFORMTHEME` env variable before staring Xorg session.

```
setenv QT_QPA_PLATFORMTHEME panda
```
