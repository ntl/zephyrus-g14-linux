# Pop!_OS Installation Walkthrough

## Known Issues

TODO

## Prerequisites

In order to install Pop onto the G14, an installation disk image (abbreviated as "ISO") will need to be downloaded and burned onto an installation media, such as a USB flash drive. Download the installation ISO from the [Pop website](https://pop.system76.com). Even though the laptop has an NVIDIA GPU, choose the non-NVIDIA ISO. The drivers for the GPU will be installed later in this guide.

Pop's download page also includes a SHA256 sum, which can be used to verify that the ISO file downloaded locally precisely matches the installation disk image that System76 officially published. On Linux, the checksum can be verified via the `sha256sum` command:

    > sha256sum ~/Downloads/pop-os_20.04_amd64_intel_11.iso
    258d88478cd42556a2646e5d93d46fb18366a261cbd24b684fc4239490d61ff5  /home/user/Downloads/pop-os_20.04_amd64_intel_11.iso

The above checksum beginning with `258d8847` can be verified visually against the SHA256 checksum on Pop's download page. If using Windows or MacOS, the graphical tool used to burn the ISO file into the installation media should also present the checksum.

## Installation

### Boot from Installation Media

With the installation media connected to the G14, turn the G14 on and hit ESC a few times when the "Republic of Gamers" logo appears. This will bring up the boot menu, allowing the choice of either booting from the internal SSD (which comes with Windows 10 pre-installed) or from the installation media. If the installation media cannot be selected, then there is either a problem with the media itself, or with the burning process.

Since, at present, Pop 20.04 does not support the G14 out of the box, a special parameter must be set in order to boot into the installation media. After selecting the installation media in the BIOS boot menu, press the space bar as soon as the Pop splash screen appears. A countdown timer shown at the bottom of the screen will disappear. Press `e` to access a text editor that will allow a custom boot parameter to be specified. Initially, the editor will show text that looks like this:

```
setparams 'Try or Install Pop_OS'

	set gfxpayload=keep
	linux /casper_pop-os_20.04_amd64_intel_debug_20/vmlinuz.efi boot=casper live-media-path=/casper_pop-os_20.04_amd64_intel_debug_20 hostname=pop-os username=pop-os noprompt  ---
	initrd /casper_pop-os_20.04_amd64_intel_debug_20/initrd.gz
```

The cursor will be located at the very first character (the `s` in `setparams` on the first line), and the arrow keys can move the cursor around the text. Navigate to the line that begins with `linux`, and then move to the trailing `---` at the end, either with the right arrow key or by pressing `CTRL-e`. Delete the three hyphens and replace them with the following text:

    nomodeset amdgpu.exp_hw_support modprobe=blacklist-nouveau

Afterward, the editor should show text that looks like this:

```
setparams 'Try or Install Pop_OS'

	set gfxpayload=keep
	linux /casper_pop-os_20.04_amd64_intel_debug_20/vmlinuz.efi boot=casper live-media-path=/casper_pop-os_20.04_amd64_intel_debug_20 hostname=pop-os username=pop-os noprompt nomodeset amdgpu.exp_hw_support modprobe=blacklist-nouve\
au
	initrd /casper_pop-os_20.04_amd64_intel_debug_20/initrd.gz
```

When the kernel parameters have been entered, press `F10` to boot into the installation media. After a few minutes of scrolling text, the graphical installation utility should appear.

### Graphical Installation

Select the desired language and keyboard layout, and then choose between either erasing the existing operating system entirely and replacing it with Pop (labeled "Clean Install") or a more advanced installation with a custom partition scheme (labeled "Custom (Advanced)"). The remainder of the guide will assume "Clean Install" was selected, and that an encryption password was subsequently supplied. If a dual boot setup with Windows is needed, it can be installed alongside a clean installation after Pop is up and running. Note: disk encryption cannot be configured during a custom installation.

After either choosing a clean install, or initiating a custom installation, the installation will commence and take a few minutes to complete. Once it is done, the graphical installation tool will prompt to restart. **Do not restart just yet!** Before rebooting, some manual configuration is necessary, otherwise Pop will not be able to boot.

Press the windows key, which will cause the background to dim, and a set of icons to appear on the left side of the screen (similar to the application dock on MacOS). Click on the icon that looks like a computer monitor, which opens a terminal. The prompt should look like this:

```
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

pop-os@pop-os:~$ █
```

### Configure Boot Parameters

The terminal is opened to the `pop-os` user. The following changes need to be made by the superuser ("root") account, so switch to the root user with `sudo su -`:

    pop-os@pop-os:~$ sudo su -
    root@pop-os:~# █

Pop's installation set up a small boot partition at the beginning of the hard drive. It contains a file that must be edited before the laptop can successfully boot into Pop. First, create a _mount_ point on the filesystem at `/media/boot` where the boot partition will be made available for editing:

    root@pop-os:~# mkdir /media/boot
    root@pop-os:~# █

Mount the boot partition (the name of the hard drive device is `nvme0n1p1`):

    root@pop-os:~# mount /dev/nvme0n1p1 /media/disk
    root@pop-os:~# █

Edit the file located at `/media/boot/loader/entries/Pop_OS-current.conf` with `nano` (or `vi` for users familiar with Vim):

    root@pop-os:~# nano /media/boot/loader/entries/Pop_OS-current.conf

Press the down arrow key a few times to navigate to the bottom of the file, on the line of text beginning with `options`. Navigate to the end of the line by either pressing the arrow key repeatedly, or by pressing `CTRL-e` once. After `splash`, add `nomodeset amdgpu.exp_hw_support modprobe.blacklist-nouveau` similar to before, when booting into the installation media. Save the file with `CTRL-o` and Enter, then exit the text editor (Nano) with `CTRL-x`. Next, close the terminal by clicking on the `X` in the top right corner of the window (it is safe to click "Close Terminal" when prompted).

Now, click "Restart Device" and wait for the system to reboot. If an encryption password was given during installation, it must be supplied before the operating system can finish booting.

A welcome dialog should appear next, which allows the installation process to be completed.

### Finish Pop_OS Installation

Click "Next" to resume installation, selecting a keyboard layout, a WiFi network, location services, timezone, and any online accounts that Pop can be connected to. Finally, set up a primary user account and then click "Start Using Pop_OS" on the final screen that indicates installation is complete.

Now that Pop's installation procedure has finished, the operating system needs a few more changes to be made from a terminal. As before, press the Windows key and click on the terminal icon on the left menu bar. Then use `sudo su -` to gain superuser ("root") access (it will prompt for a password this time):

```
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

YOUR-USER-NAME@pop-os:~$ sudo su -
[sudo] password for YOUR-USER-NAME:
root@pop-os:~# █
```

#### Disable Open Source NVIDIA Driver (nouveau)

The open source driver for NVIDIA chips, `nouveau`, does not work with recent NVIDIA offerings, including the GPUs included with the G14. It must be disabled permanently by creating a configuration file for `modprobe` (which is Linux's tool for controlling device drivers). Configuration files for modprobe go in `/etc/modprobe.d`. Edit `/etc/modprobe.d/disable-nvidia-nouveau.conf` with Nano:

    root@pop-os:~# nano /etc/modprobe.d/disable-nvidia-nouveau.conf

Type the following into the editor, and save and exit (`CTRL-o` and `CTRL-x`, respectively):

    blacklist nouveau
    options nouveau modeset=0

#### Make Kernel Boot Parameters Permanent

The kernel boot parameters configured earlier, `nomodeset amdgpu.exp_hw_support modprobe.blacklist-nouveau`, need to be made permanent so that Pop's updates are free to refresh the boot loader configuration when necessary. Edit `/etc/kernelstub/configuration`:

    root@pop-os:~# nano /etc/kernelstub/configuration

The file should look like this:

```
{
  "default": {
    "kernel_options": [
      "quiet",
      "splash"
    ],
    "esp_path": "/boot/efi",
    "setup_loader": false,
    "manage_mode": false,
    "force_update": false,
    "live_mode": false,
    "config_rev": 3
  },
  "user": {
    "kernel_options": [
      "quiet",
      "loglevel=0",
      "systemd.show_status=false",
      "splash"
    ],
    "esp_path": "/boot/efi",
    "setup_loader": true,
    "manage_mode": true,
    "force_update": false,
    "live_mode": false,
    "config_rev": 3
  }
}
```

Under _both_ `"kernel_options"` entries, navigate to the final line (`"splash"`) and add the boot parameters `nomodeset`, `amdgpu.exp_hw_support`, and `modprobe.blacklist-nouveau` one at a time, giving each its own line. Also, add a comma after `"splash"`. The sections should both look like this afterward:

```
    ...
    "kernel_options": [
      "quiet",
      ...
      "splash",
      "nomodeset",
      "amdgpu.exp_hw_support",
      "modprobe.blacklist-nouveau"
    ],
    "esp_path": "/boot/efi",
    ...
```

When this is complete, save and exit (`CTRL-o` and `CTRL-x`, respectively). Now run `kernelstup` to cause the changes to `/etc/kernelstub/configuration` take into effect:

    root@pop-os:~# kernelstub
    root@pop-os:~# █

At this point, the laptop should be able to boot on its own without any intervention. Click on the battery icon at the top right of the screen, then "Power Off / Log Out", then "Power Off...". A modal dialog should appear; click "Restart."

## Custom Kernel (Xanmod)

A recent kernel (5.8 or newer) is needed in order to take full advantage of the G14's hardware. Open Firefox by pressing the Windows key and clicking on the Firefox icon, which is at the top of the left menu bar. Navigate to [https://xanmod.org](https://xanmod.org). Find the section titled Install via AptURL, near the top of the page. First, the Xanmod repositories have to be added to the Pop installation. Where the page indicates to "First install the XanMod Repository Setup, then select a branch," click on the "XanMod Repository Setup" link. Firefox will open a file download dialog, allowing a file called `xanmod-repository.deb` to be installed with Eddy, which is the default program for installing `.deb` files. Click `OK` and then "Install" on the dialog that appears. Enter the user account password when prompted, and wait for the repository setup to finish. Eddy may be closed afterwards.

Open a Terminal, gain superuser privileges with `sudo su -`, and install `linux-xanmod-edge` with `apt`:

```
YOUR-USER-NAME@pop-os:~$ sudo su -
root@pop-os:~# sudo apt install linux-xanmod-edge
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following additional packages will be installed:
  libelf-dev linux-headers-5.8.1-xanmod1 linux-image-5.8.1-xanmod1
Suggested packages:
  preload gamemode
The following NEW packages will be installed:
  libelf-dev linux-headers-5.8.1-xanmod1 linux-image-5.8.1-xanmod1
  linux-xanmod-edge
0 upgraded, 4 newly installed, 0 to remove and 342 not upgraded.
Need to get 77.4 MB of archives.
After this operation, 409 MB of additional disk space will be used.
Do you want to continue? [Y/n] █
```

Type `Y` to continue, and eventually the installation will finish. The final few lines of text will look like this:

```
...
kernelstub.Installer : INFO     Making entry file for Pop_OS
Setting up libelf-dev:amd64 (0.176-1.1build1) ...
Setting up linux-xanmod-edge (5.8.1-xanmod1-0) ...
root@pop-os:~# █
```

Reboot again, and the laptop should boot into a 5.8 kernel patched by Xanmod.

Next, the device needs to install some software specialized to provide support for the G14. Open a new terminal and create a directory, `g14`, where this software will be downloaded and installed from:

```
YOUR-USER-NAME@pop-os:~$ mkdir g14
YOUR-USER-NAME@pop-os:~$ cd g14
YOUR-USER-NAME@pop-os:~/g14$ █
```

### hid-asus-rog (G14 Keyboard Support)

First, install the kernel patch from [this project](https://gitlab.com/asus-linux/hid-asus-rog), which will make the G14 specific keys on the keyboard available. Download it using `git`:

```
YOUR-USER-NAME@pop-os:~/g14$ git clone --depth -1 https://gitlab.com/asus-linux/hid-asus-rog.git
Cloning into 'hid-asus-rog'...
remote: Enumerating objects: 10, done.
remote: Counting objects: 100% (10/10), done.
remote: Compressing objects: 100% (10/10), done.
remote: Total 10 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (10/10), 21.16 KiB | 1.17 MiB/s, done.
```

Enter the `hid-asus-rog` directory and install with `sudo make dkms` and `sudo make onboot`:

```
YOUR-USER-NAME@pop-os:~/g14$ cd hid-asus-rog
```
<!-- separator -->

```
YOUR-USER-NAME@pop-os:~/g14/hid-asus-rog$ sudo make dkms
[sudo] password for YOUR-USER-NAME:
make -C /lib/modules/5.8.1-xanmod1/build M=/home/YOUR-USER-NAME/g14/hid-asus-rog modules
make[1]: Entering directory '/usr/src/linux-headers-5.8.1-xanmod1'
  CC [M]  /home/YOUR-USER-NAME/g14/hid-asus-rog/src/hid-asus-rog.o
  MODPOST /home/YOUR-USER-NAME/g14/hid-asus-rog/Module.symvers
  CC [M]  /home/YOUR-USER-NAME/g14/hid-asus-rog/src/hid-asus-rog.mod.o
  LD [M]  /home/YOUR-USER-NAME/g14/hid-asus-rog/src/hid-asus-rog.ko
make[1]: Leaving directory '/usr/src/linux-headers-5.8.1-xanmod1'
```
<!-- separator -->
```
YOUR-USER-NAME@pop-os:~/g14/hid-asus-rog$ sudo make onboot
[sudo] password for YOUR-USER-NAME:
echo "blacklist hid-asus" > /etc/modprobe.d/asus-rog.conf
```

Leave the `hid-asus-rog` with `cd ..`

```
YOUR-USER-NAME@pop-os:~/g14/hid-asus-rog$ cd ..
YOUR-USER-NAME@pop-os:~/g14$ █
```

### asus-rog-nb-wmi (Windows Media Interface support)

Next, install another kernel patch, this time from [this project](https://gitlab.com/asus-linux/asus-rog-nb-wmi), which adds support for the G14 keyboard backlight control. As before, with `hid-asus-rog`, first download the project with `git` and enter it with `cd`:

```
YOUR-USER-NAME@pop-os:~/g14$ git clone --depth -1 https://gitlab.com/asus-linux/asus-rog-nb-wmi.git
Cloning into 'asus-rog-nb-wmi'...
...
Unpacking objects: 100% (10/10), 6.04 KiB | 1.21 MiB/s, done.
YOUR-USER-NAME@pop-os:~/g14$ cd asus-rog-nb-wmi
YOUR-USER-NAME@pop-os:~/g14/asus-rog-nb-wmi$ █
```

Install with `sudo make dkms` and `sudo make onboot`:

```
YOUR-USER-NAME@pop-os:~/g14/asus-rog-nb-wmi$ sudo make dkms
...
```
<!-- separator -->
```
YOUR-USER-NAME@pop-os:~/g14/asus-rog-nb-wmi$ sudo make onboot
...
```

Before proceeding, ensure that the above kernel patches are active with `dkms status`:

```
YOUR-USER-NAME@pop-os:~/g14/asus-rog-nb-wmi$ dkms status
asus-rog-nb-wmi, 0.1, 5.8.1-xanmod1, x86_64: installed
hid-asus-rog, 0.2, 5.8.1-xanmod1, x86_64: installed
...
```

### Screen Brightness

TODO:  0002-drm-amd-display-use-correct-scale-for-actual_brightness.patch

### asus-nb-ctrl (Asus Notebook Control)

Next, install `asus-nb-ctrl` from [this project](https://gitlab.com/asus-linux/asus-nb-ctrl), which is a program that controls the G14 hardware, e.g. turning keyboard backlight on and off, setting the CPU performance mode, etc. Download the project with `git` and enter it with `cd`:

```
YOUR-USER-NAME@pop-os:~/g14$ git clone --depth -1 https://gitlab.com/asus-linux/asus-nb-ctrl.git
Cloning into 'asus-nb-ctrl'...
...
Unpacking objects: 100% (10/10), 6.04 KiB | 1.21 MiB/s, done.
YOUR-USER-NAME@pop-os:~/g14$ cd asus-nb-ctrl
YOUR-USER-NAME@pop-os:~/g14/asus-nb-ctrl$ █
```

Before the project can be compiled, [Rust](https://rust-lang.org) and a few other dependencies must be installed:

```
YOUR-USER-NAME@pop-os:~/g14/asus-nb-ctrl$ sudo apt install rustc libusb-1.0-0-dev libdbus-1-dev llvm libclang-dev libudev-dev
```

Compile the package with `make`:

```
YOUR-USER-NAME@pop-os:~/g14/asus-nb-ctrl$ make
```

Install with `sudo make install`:

```
YOUR-USER-NAME@pop-os:~/g14/asus-nb-ctrl$ sudo make install
[sudo] password for YOUR-USER-NAME:
```

After that, reboot, and the kernel should be all set. The keyboard backlight as well as the screen brightness should be controllable after logging in.

## Getting the Remaining Hardware Working

### Update the System

Click on the clock at the top middle of the screen. There should be a notification that operating system updates are available. Click through this process and allow it to finish. Another restart is recommended after the update finishes.

### Audio

See: https://asus-linux.org/wiki/rog-zephyrus/g14-and-g15/hardware/audio/

As the superuser, edit `/usr/share/pulseaudio/alsa-mixer/paths/analog-output-speaker.conf` and find the part of the file that contains the sections titled `[Element Hardware Master]`, `[Element Master]`, and `[Element Master Mono]`. Adjust those sections so that they look like this (from the document just linked):

```
[Element Hardware Master]
switch = mute
volume = ignore

[Element Master]
switch = mute
volume = ignore

[Element PCM]
switch = mute
volume = merge
override-map.1 = all
override-map.2 = all-left,all-right
```

Run `pulseaudio -k` as a normal (non-super) user. The volume buttons should now function properly.

### Graphics

The G14 has two GPUs -- the integrated AMD Vega GPU ("iGPU") which shares the same die as the CPU, and a separate dedicated NVIDIA GPU ("dGPU"). Neither is correctly configured just yet. To configure the iGPU, edit `/usr/share/X11/xorg.conf.d/90-zephyrus-g14.conf`:

```
Section "Device"
	Identifier "iGPU"
	Driver "amdgpu"
EndSection

Section "Screen"
	Identifier "iGPU"
	Device "iGPU"
EndSection
```

This file will be edited later, after the NVIDIA drivers are installed.

### Touchpad

TODO

### Battery Life and Performance

TODO
