Arch Installation Walkthrough
=============================

[Arch Linux](https://archlinux.org) does not include an installer program that automates installation, and instead relies on a [manual installation procedure](https://wiki.archlinux.org/index.php/Installation_guide). For users new to Arch, this can present a high degree of difficulty -- while Arch allows a good deal of customization of all aspects of the operating system, that flexibility comes at the expense of forcing beginners to confront decisions that they are usually unprepared to make. However, at the moment, Arch is far and away the best supported distribution for G14, so the installation process is documented here, and in far greater detail than would make sense for the official Arch documentation. With that in mind, this walkthrough makes quite a few choices on behalf of the user:

1. Encrypted root filesystem
2. GNOME desktop environment
3. Graphical boot process

Finally, the walkthrough also installs a customized version of the Linux kernel that contains support for the G14 not yet merged upstream in the official Linux kernel.

Known Issues
------------
* An external monitor connected to the USB Type-C port requires the dGPU (NVidia) to drive the whole desktop, which significantly reduces battery life
* Switching the desktop between the dGPU (NVidia) and iGPU (AMD) requires restarting the desktop
* The graphical boot program, Plymouth, cannot make use of an external display connected to USB Type-C port

Prerequisites
-------------
In order to install Arch onto the G14, an Arch ISO will need to be downloaded and burned onto an installation media, such as a USB flash drive. It is also a good idea to skim through Arch's [Installation Guide](https://wiki.archlinux.org/index.php/Installation_guide) before beginning.

Download the Arch installation ISO from the Arch [download page](https://www.archlinux.org/download/). Verify the checksum before proceeding to ensure the download wasn't compromised:

    > md5sum archlinux-2020.06.01-x86_64.iso 
    1507b54b248d8be244ed6820baa04aeb  archlinux-2020.06.01-x86_64.iso

The MD5 checksum must be compared against the one on the official Arch download page, linked above. It is possible that an attacker has also compromised that download page, so if this is a concern, use GPG to verify that the ISO was produced by a trusted Arch contributor. To burn the ISO onto an installation media, use a USB flashing tool such as [Popsicle](https://github.com/pop-os/popsicle).

If possible, consider connecting the G14 to a wired internet connection via an USB Ethernet dongle or dock. This will likely increase the performance and reliability of the installation process, as it has to download the entire operating system over the internet.

The guide assumes basic familiarity with Linux, like the ability to run commands in a shell prompt, as well as basic use of the `nano` text editor.

Credits
-------
Linux on the Zephyrus G14 would not be possible without the efforts from the maintainers of the following projects:

* [asus-rog-zephyrus-g14](https://lab.retarded.farm/zappel/asus-rog-zephyrus-g14/)
* [rog-core](https://github.com/flukejones/rog-core)

Installation
------------

### Boot From Installation Media

With the installation media connected to the G14, turn the G14 on and hit ESC a few times when the "Republic of Gamers" logo appears. This will bring up the boot menu, allowing the choice of either booting from the internal SSD (which comes with Windows 10 pre-installed) or from the installation media. If the installation media cannot be selected, then there is either a problem with the media itself, or with the burning process.

Booting from the installation media will bring up a menu with a few options:

    Arch Linux archiso x86_64 UEFI CD
    UEFI Shell (Full) x86_64
    UEFI Shell x86_64
    EFI Default Loader
    Reboot Into Firmware Interface

The first option, `Arch Linux archiso x86_64 UEFI CD`, is selected by default. Hit ENTER to select it and proceed. The installation media contains a live Arch environment, and does not immediately deliver the user into an installation process. If a command prompt is visible, `root@archiso ~ #`, then the boot process has completed successfully.

Note: some users have reported needing to add a kernel command line parameter when booting from the ISO. If the ISO boot fails, restart the laptop, and when the above menu appears, press the `e` key. A line of text should appear below the menu:

    initrd=\EFI\archiso\intel_ucode.img initrd=\EFI\archiso\amd_ucode.img initrd=\

Press and hold the right arrow key until the end of the line is reached. The bottom line of text should now look like this, with the cursor appearing at the end:

    g initrd=\EFI\archiso\archiso.img archisobasedir=arch archisolabel=ARCH_202006

Add `nouveau.modeset=0` to the end of the line and then press ENTER.

    iso\archiso.img archisobasedir=arch archisolabel=ARCH_202006 nouveau.modeset=0

### Configure Disk Partitions

To configure partitions on the G14's internal SSD, run `gdisk /dev/nvme0n1`. Output should look similar to this: 

    root@archiso ~ # gdisk /dev/nvme0n1
    GPT fdisk (gdisk) version 1.0.5

    The protective MBR's 0xEE partition is oversized! Auto-repairing.

    Partition table scan:
      MBR: protective
      BSD: not present
      APM: not present
      GPT: present

    Found valid GPT with protective MBR; using GPT

    Command (? for help):

Delete any and all partitions. For example: 

    Command (? for help): d
    Partition number (1-4): 1

    Command (? for help): d
    Partition number (2-4): 2

    Command (? for help): d
    Partition number (3-4): 3

    Command (? for help): d
    Using 4

    Command (? for help):

Create an EFI boot partition with a size of 512MB:

    Command (? for help): n
    Partition number (1-128, default 1): 1
    First sector (34-2000409230, default = 2048) or {+-}size{KMGTP}:
    Last sector (2048-2000409230, default = 2000409230) or {+-}size{KMGTP}: +512M
    Current type is 8300 (Linux filesystem)
    Hex code or GUID (L to show codes, Enter = 8300): L
    Type search string, or <Enter> to show all codes: EFI
    ef00 EFI system partition
    Hex code or GUID (L to show codes, Enter = 8300): ef00
    Changed type of partition to 'EFI system partition'

    Command (? for help)

Next, create a Linux filesystem that occupies the remainder of the disk:

    Command (? for help): n
    Partition number (2-128, default 2): 2
    First sector (34-2000409230, default = 1050624) or {+-}size{KMGTP}:
    Last sector (1050624-2000409230, default = 2000409230) or {+-}size{KMGTP}: -0
    Current type is 8300 (Linux filesystem)
    Hex code or GUID (L to show codes, Enter = 8300): 8300
    Changed type of partition to 'Linux filesystem'

    Command (? for help)

Persist the changes to the disk partitioning (NOTE: from this point onward, the previous operating system, such as Windows 10, will be erased):

    Command (? for help): w

    Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
    PARTITIONS!!

    Do you want to proceed? (Y/N): Y
    OK: writing new GUID partition table (GPT) to /dev/nvme0n1.
    The operation has completed successfully.
    root@archiso ~ #

The EFI partition, `/dev/nvme0n1p1`, will be mounted at `/boot`, and will not be encrypted. Format it as FAT32:

    root@archiso ~ # mkfs.fat -F32 /dev/nvme0n1p1
    mkfs.fat 4.1 (2017-01-24)
    root@archiso ~ #

The Linux filesystem partition, `/dev/nvme0n1p2`, will be mounted at `/`, but will be encrypted. To set up encryption:

    root@archiso ~ # cryptsetup -y -v luksFormat /dev/nvme0n1p2

    WARNING!
    ========
    This will overwrite data on /dev/nvme0n1p2 irrevocably.

    Are you sure? (Type 'yes' in capital letters): YES
    Enter passphrase for /dev/nvme0n1p2: YOUR-FILESYSTEM-ENCRYPTION-PASSWORD
    Verify passphrase: YOUR-FILESYSTEM-ENCRYPTION-PASSWORD
    WARNING: Locking directory /run/cryptsetup is missing!
    Key slot 0 created.
    Command successful.
    cryptsetup -y -v luksFormat /dev/nvme0n1p2  29.49s user 1.72s system 31% cpu 1:39.50 total
    root@archiso ~ #

Next, make the encrypted filesystem available to the operating system:

    root@archiso ~ # cryptsetup open /dev/nvme0n1p2 cryptroot
    Enter passphrase for /dev/nvme0n1p2: YOUR-FILESYSTEM-ENCRYPTION-PASSWORD
    cryptsetup open /dev/nvme0n1p2 cryptroot  6.25s user 0.26s system 101% cpu 6.396 total
    root@archiso ~ #

The encrypted partition can now be configured as a Linux filesystem:

    root@archiso ~ # mkfs.ext4 /dev/mapper/cryptroot
    Creating filesystem with 249915729 4k blocks and 62480384 inodes
    Filesystem UUID: 00000001-0000-4000-8000-000000000000
    Superblock backups stored on blocks:
            32768, 98304, *omitted*
            4097000, 7962624, *omitted*
            102400000, 214990848, *omitted*

    Allocating group tables: done
    Writing inode tables: done
    Creating journal (262144 blocks): done
    Writing superblocks and filesystem accounting information: done

    root@archiso ~ #

Before the operating system can be installed, the filesystems that were just created need to be mounted within the `/mnt` directory:

    root@archiso ~ # mount /dev/mapper/cryptroot /mnt
    root@archiso ~ # mkdir /mnt/boot
    root@archiso ~ # mount /dev/nvme0n1p1 /mnt/boot

### Prepare for Installation

If the G14 isn't connected to the internet via an Ethernet cable, configure wireless networking with `wifi-menu`. For networks that don't use DHCP, manual configuration if the IP address, gateway, etc. is required. Verify that the G14 is connected to the internet via `ping archlinux.org`:

    root@archiso ~ # ping -c 4 archlinux.org
    PING archlinux.org (138.201.81.199) 56(84) bytes of data.
    64 bytes from apollo.archlinux.org (138.201.81.199): icmp_seq=1 ttl=47 time=111ms
    64 bytes from apollo.archlinux.org (138.201.81.199): icmp_seq=2 ttl=47 time=111ms
    64 bytes from apollo.archlinux.org (138.201.81.199): icmp_seq=3 ttl=47 time=111ms
    64 bytes from apollo.archlinux.org (138.201.81.199): icmp_seq=4 ttl=47 time=111ms

To configure Arch's installation to use a nearby mirror, edit the `/etc/pacman.d/mirrorlist` file and move the nearest mirror to the top (in Nano, CTRL-K will cut the current line of text, and CTRL-U will paste the line of text that was previously cut):

    root@archiso ~ # nano /etc/pacman.d/mirrorlist

Press CTRL-O, then ENTER, to save the file, then press CTRL-X to quit Nano.

### Add G14-Specific Pacman Repository

A third party package repository is provided by the Linux G14 community, and includes custom kernels and graphics drivers that are tested with the G14. To make the repository available to `pacman`, edit `/etc/pacman.conf` and add the following text just above the section titled `[core]`:

    [g14]
    SigLevel = Optional TrustAll
    Server = https://arch.retarded.farm

Afterwards, the relevant section should look like this:

    *omitted*

    # The testing repositories are disabled by default. To enable, uncomment the
    # repo name header and Include lines. You can add preferred servers immediately
    # after the header, and they will be used before the default mirrors.

    #[testing]
    #Include = /etc/pacman.d/mirrorlist

    [g14]
    SigLevel = Optional TrustAll
    Server = https://arch.retarded.farm

    [core]
    Include = /etc/pacman.d/mirrorlist

    [extra]
    Include = /etc/pacman.d/mirrorlist

    *omitted*

Alternatively, advanced users can manually install the packages available on the community repository.

### Install Arch

The base operating system packages can be installed via the `pacstrap` utility:

    root@archiso ~ # pacstrap /mnt base linux-g14 linux-g14-headers linux-firmware amd-ucode nano

After `pacstrap` finishes, the filesystem partitions created earlier need to be configured:

    root@archiso ~ # genfstab -U /mnt >> /mnt/etc/fstab

Also, if the G14 community repository was added earlier, it must now also be added to `/mnt/etc/pacman.conf`, following the same instructions (but varying the file name to include `/mnt` at the beginning).

At this point, the new installation can be entered via `arch-chroot`:

    root@archiso ~ # arch-chroot /mnt
    [root@archiso /]# 

### Configure Arch

#### Localization

Set the clock to the local timezone (use the tab key after `/usr/share/zoneinfo` if needed):

    [root@archiso /]# ln -svf /usr/share/zoneinfo/America/Chicago /etc/localtime

Configure the system clock:

    [root@archiso /]# hwclock --systohc

Configure the locale by editing `/etc/locale.gen`. Remove the leading comment character, `#`, from any lines that pertain to a locale that will be used (such as `en_US` or `de_DE`). To search for a specific locale, such as `en_US`, press CTRL-W, then the locale name, then press ENTER. The next time CTRL-W is pressed, the previous search term can be used again by pressing ENTER immediately after CTRL-W. Afterwards, run `locale-gen`.

    [root@archiso /]# nano /etc/locale.gen
    [root@archiso /]# locale-gen
    Generating locales...
      en_US.UTF-8... done
      en_US.ISO8859-1... done
    Generation complete.
    [root@archiso /]# 

Set the `LANG` variable in `/etc/locale.conf` to the default locale, e.g. `en_US.UTF-8`, and then export the `LANG` variable for the local shell session:

    [root@archiso /]# echo LANG=en_US.UTF-8 >> /etc/locale.conf
    [root@archiso /]# export LANG=en_US.UTF-8
    [root@archiso /]# 

#### Network Configuration

Set the hostname for the G14, e.g. `arch-g14`

    [root@archiso /]# echo arch-g14 > /etc/hostname 

Configure static hosts in `/etc/hosts`. Pressing ENTER right after `cat > /etc/hosts <<EOF` will allow the contents of the file to be typed in directly, without having to open `nano`. When entering the file contents, there will not be a command prompt shown at the beginning of each line (i.e. `[root@archiso /]# `). The `EOF` line will cause the file to be written, and the command prompt to return:

    [root@archiso /]# cat > /etc/hosts <<EOF
    127.0.0.1 localhost
    ::1 localhost
    127.0.1.1 arch-g14
    EOF
    [root@archiso /]#

In the event there was a mistake typing in the above file, the file can be opened in Nano with `nano /etc/hosts`.

#### Recreate initramfs Image

Since the root filesystem is encrypted, a new `initramfs` image must be created with `mkinitcpio`. However, encryption must first be added to `/etc/mkinitcpio.conf`. Edit the file and find where `HOOKS` is set. It should look like this:

    HOOKS=(base udev autodetect modconf block filesystems keyboard fsck)

Add `encrypt` between `block` and `filesystems`, and optionally add `keymap` after `keyboard` if an alternate keyboard layout is necessary to type in the decryption password. Afterward, it should look like this:

    HOOKS=(base udev autodetect modconf block encrypt filesystems keyboard keymap fsck)

Now run `mkinitcpio -P` and ensure the build hooks `encrypt` and `keymap` (if added above) are run:

    [root@archiso /]# mkinitcpio -P
    ==> Building image from preset: /etc/mkinitcpio.d/linux.preset 'default'
      -> -k /boot/vmlinuz-linux -c /etc/mkinitcpio.conf -g /boot/initramfs-linux.img
    ==> Starting build: 5.7.5-arch1-1
      -> Running build hook: [base]
      -> Running build hook: [udev]
      -> Running build hook: [autodetect]
      -> Running build hook: [modconf]
      -> Running build hook: [block]
      -> Running build hook: [encrypt]
      -> Running build hook: [filesystems]
      -> Running build hook: [keyboard]
      -> Running build hook: [keymap]
      -> Running build hook: [fsck]
    ==> Generating module dependencies
    ==> Creating gzip-compressed initcpio image: /boot/initramfs-linux.img
    ==> Image generation successful
      -> Building image from preset: /etc/mkinitcpio.conf -g /boot/initramfs-linux-fallback.img -S autodetect
    ==> Starting build: 5.7.5-arch1-1
      -> Running build hook: [base]
      -> Running build hook: [udev]
      -> Running build hook: [modconf]
      -> Running build hook: [block]
    ==> WARNING: Possibly missing firmware for module: wd719x
    ==> WARNING: Possibly missing firmware for module: aic94xx
      -> Running build hook: [encrypt]
      -> Running build hook: [filesystems]
      -> Running build hook: [keyboard]
      -> Running build hook: [keymap]
      -> Running build hook: [fsck]
    ==> Generating module dependencies
    ==> Creating gzip-compressed initcpio image: /boot/initramfs-linux-fallback.img
    ==> Image generation successful
    [root@archiso /]#

Note: If a graphical boot process is desired (described later in this guide, under "Graphical Boot Process"), the preceding command isn't strictly necessary, as it will be invoked again when the graphical boot is set up.

#### Install Bootloader (systemd-boot)

To install `systemd-boot` run `bootctl install`:

    [root@archiso /]# bootctl install
    Created "/boot/EFI"
    Created "/boot/EFI/systemd"
    Created "/boot/EFI/BOOT"
    Created "/boot/loader"
    Created "/boot/loader/entries"
    Created "/boot/EFI/Linux"
    Copied "/usr/lib/systemd/boot/efi/systemd-bootx64.efi" to "/boot/EFI/systemd/systemd-bootx64.efi"
    Copied "/usr/lib/systemd/boot/efi/systemd-bootx64.efi" to "/boot/EFI/BOOT/BOOTX64.EFI"
    Random seed file /boot/loader/random-seed successfully written (512 bytes)
    Created EFI boot entry "Linux Boot Manager".
    [root@archiso /]#

The boot loader is configured by writing `/boot/loader/loader.conf`. The file already exists, but replace it with this:

    [root@archiso /]# cat > /boot/loader/loader.conf <<EOF
    default arch-g14.conf
    timeout 0
    EOF
    [root@archiso /]#

For more information about configuring the boot loader, see the [documentation page](https://wiki.archlinux.org/index.php/Systemd-boot#Configuration)

Note for advanced users: if the G14 community repository wasn't added to `/etc/pacman.conf` earlier, then the following configuration files should refer to the stock Arch kernel, not the G14 one (as it isn't installed yet).

Next, add an entry for the stock Arch kernel. Later, when compiling custom kernels to support the full breadth of G14 hardware, additional entries will be added for each custom kernel. For now, the boot loader entry that will be edited is `/boot/loader/entries/arch.conf`. Write the entry:

    [root@archiso /]# cat > /boot/loader/entries/arch-g14.conf <<EOF
    title Arch Linux (G14)
    linux /vmlinuz-linux-g14
    initrd /amd-ucode.img
    initrd /initramfs-linux-g14.img
    EOF
    [root@archiso /]#

The entry will require additional configuration in order to decrypt the root partition, specifically the unique identifier ("UUID") of the encrypted partition. To get the UUID, run the following command:

    [root@archiso /]# blkid | grep /dev/nvme0n1p2 | cut -f 2 -d ' ' | sed 's/"//g'
    UUID=00000001-0000-4000-8000-000000000000
    [root@archiso /]#

Note: the above UUID, `00000001-0000-4000-8000-000000000000`, is just an example. The actual UUID will be a random string of text, and look like `2207898c-c6e1-4e5d-b636-2b9aa9521a5c`.

To make the above UUID available to the boot loader entry file, append the output of the `blkid` previously run to the file itself:

    [root@archiso /]# blkid | grep /dev/nvme0n1p2 | cut -f 2 -d ' ' | sed 's/"//g' >> /boot/loader/entries/arch-g14.conf
    [root@archiso /]#

Edit `/boot/loader/entries/arch-g14.conf` with `nano` and change `UUID=00000001-0000-4000-8000-000000000000` to this:

    options root=/dev/mapper/cryptroot cryptdevice=UUID=00000001-0000-4000-8000-000000000000:cryptroot rw trace_clock=local

Run `bootctl list` to verify that the entry just added will function:

    [root@archiso /]# bootctl list
    Boot Loader Entries:
            title: Arch Linux G14 (default)
               id: arch.conf
           source: /boot/loader/entries/arch-g14.conf
            linux: /vmlinuz-linux
           initrd: /amd-ucode.img
                   /initramfs-linux-g14.img

    *omitted*
    [root@archiso /]#

#### Install Desktop Environment

This step assumes a GNOME desktop environment is desired. For any other environment, such as KDE, a different set of packages will need to be installed.

    [root@arch-g14]# pacman --sync --noconfirm base-devel bluez bluez-utils chrome-gnome-shell \
      clang curl cups git gnome gnome-software-packagekit-plugin gnome-tweak-tool man man-db man-pages \
      pacman-contrib pulseaudio pulseaudio-alsa sane sudo system-config-printer systemd-swap tar texinfo \
      ttf-dejavu xdg-utils xf86-video-amdgpu xorg xorg-server

After all packages are installed, enable the system services associated with the above packages:

<!-- Separator -->

    [root@archiso /]# systemctl enable gdm.service
    Created symlink /etc/systemd/system/display-manager.service → /usr/lib/systemd/system/gdm.service

<!-- Separator -->

    [root@archiso /]# systemctl enable NetworkManager.service
    Created symlink /etc/systemd/system/multi-user.target.wants/NetworkManager.service → /usr/lib/systemd/system/NetworkManager.service
    Created symlink /etc/systemd/system/dbus-org.freedesktop.nm-dispatcher.service → /usr/lib/systemd/system/NetworkManager-dispatcher.service
    Created symlink /etc/systemd/system/network-online.target.wants/NetworkManager-wait-online.service → /usr/lib/systemd/system/NetworkManager-wait-online.service

<!-- Separator -->

    [root@archiso /]# systemctl enable systemd-swap.service
    Created symlink /etc/systemd/system/multi-user.target.wants/systemd-swap.service → /usr/lib/systemd/system/systemd-swap.service

<!-- Separator -->

    [root@archiso /]# systemctl enable bluetooth.service
    Created symlink /etc/systemd/system/dbus-org/bluez.service → /usr/lib/systemd/system/bluetooth.service
    Created symlink /etc/systemd/system/bluetooth.target.wants/bluetooth.service → /usr/lib/systemd/system/bluetooth.service

<!-- Separator -->

    [root@archiso /]# systemctl enable org.cups.cupsd.service
    Created symlink /etc/systemd/system/printer.target.wants/org.cups.cupsd.service → /usr/lib/systemd/system/org.cups.cupsd.service
    Created symlink /etc/systemd/system/sockets.target.wants/org.cups.cupsd.socket → /usr/lib/systemd/system/org.cups.cupsd.socket
    Created symlink /etc/systemd/system/multi-user.target.wants/org.cups.cupsd.path → /usr/lib/systemd/system/org.cups.cupsd.path

#### Create Non-root User

Next, create the non-root user. This user will be available to select on the graphical login screen after rebooting.

    [root@archiso /]# useradd -G audio,video,wheel -m YOUR-USER-NAME
    [root@archiso /]# passwd YOUR-USER-NAME
    New password: YOUR-USER-PASSWORD
    Retype new password: YOUR-USER-PASSWORD
    passwd: password updated successfully
    [root@archiso /]#

The user was added to the `wheel` group above, which is intended for users granted the permission to act as superusers (e.g. `root`). To allow the above user to act as root via `sudo`, edit `/etc/sudoers` and uncomment one of the two lines that begins with `# %wheel`, depending on whether a password prompt is desired.

    [root@archiso /]# nano /etc/sudoers

Finally, switch to the non-root user:

    [root@archiso /]# su - YOUR-USER-NAME
    [YOUR-USER-NAME@archiso ~]$

Test that `sudo` is able to run as the superuser:

    [YOUR-USER-NAME@archiso ~]$ sudo echo "Running as superuser"
    Running as superuser

For the remainder of this guide, many commands will require `sudo` to be prefixed to them, in order to ensure the command is granted superuser privileges.

#### A Note About Arch User Repository

The G14 requires a few community contributed packages to be installed that do not belong to official Arch package repository. Arch's community contributed packages are hosted at the [Arch User Repository](https://aur.archlinux.org), which is usually abbreviated to "AUR." Packages downloaded from AUR need to be built from source using Arch's `makepkg` utilitiy. Create a directory that will be used for staging AUR source packages:

    [YOUR-USER-NAME@archiso ~]$ mkdir aur

Change into that directory for the next few steps, as they will involve installing packages from AUR:

    [YOUR-USER-NAME@archiso ~]$ cd aur
    [YOUR-USER-NAME@archiso aur]$

Since AUR packages are built from source, the G14's 8 core processor can speed up the process by compiling source files in parallel. To enable multi-core compilation, set the `MAKEFLAGS` environment variable to either `-j16` (for 4800HS and 4900HS CPUs) or `-j8` (for 4600HS CPUs):

    [YOUR-USER-NAME@archiso aur]$ export MAKEFLAGS="-j16"

#### Rog-Core

The Windows installation that is supplied with the G14 includes a software package called "Armoury Crate" which allows for controlling the fan speed, boost, and more. `rog-core` is a command-line utility for Linux that offers similar functionality for ROG laptops. Download the source package from AUR via git:

    [YOUR-USER-NAME@archiso aur]$ git clone --depth 1 https://aur.archlinux.org/rog-core.git
    Cloning into 'rog-core'...
    remote: Enumerating objects: 6, done.
    remote: Counting objects: 100% (6/6), done.
    remote: Compressing objects: 100% (5/5), done.
    remote: Total 6 (delta 0), reused 6 (delta 0), pack-reused 0
    Unpacking objects: 100% (6/6), 1.51 KiB | 1.51 MiB/s, done.

Enter the `rog-core` directory and build/install the package via `makepkg`:

    [YOUR-USER-NAME@archiso aur]$ cd rog-core
    [YOUR-USER-NAME@archiso rog-core]$ makepkg --install --syncdeps --noconfirm --skippgpcheck
    ==> Making package: rog-core 0.12.0-1 (Fri 26 Jun 2020 10:07:00 PM CDT)
    ==> Checking runtime dependencies...
    ==> Checking buildtime dependencies...
    ==> Retrieving sources...
    *omitted*
    (1/1) downgrading rog-core                         [##########] 100%
    :: Running post-transaction hooks...
    (1/3) Reloading system manager configuration...
    (2/3) Arming ConditionNeedsUpdate...
    (3/3) Reloading system bus configuration...
    [YOUR-USER-NAME@archiso rog-core]$

<!-- Separator -->

    [YOUR-USER-NAME@archiso rog-core]$ cd ..
    [YOUR-USER-NAME@archiso aur]$

Enable the `rog-core` service with `systemctl`:

    [YOUR-USER-NAME@archiso aur]$ sudo systemctl enable rog-core.service
    Created symlink /etc/systemd/system/multi-user.target.wants/rog-core.service → /usr/lib/systemd/system/rog-core.service.
    [YOUR-USER-NAME@archiso aur]$

#### Custom Kernel (Advanced Users Only)

If the community repository wasn't added to `/etc/pacman.conf`, then a kernel must now be compiled. First, download the arch source package via git:

    [YOUR-USER-NAME@archiso aur]$ git clone --depth 1 https://aur.archlinux.org/linux-g14.git
    Cloning into 'linux-g14'...
    remote: Enumerating objects: 12, done.
    remote: Counting objects: 100% (12/12), done.
    remote: Compressing objects: 100% (12/12), done.
    remote: Total 12 (delta 0), reused 7 (delta 0), pack-reused 0
    Unpacking objects: 100% (12/12), 67.93 KiB | 470.00 KiB/s, done.

Enter the `linux-g14` directory and then build and install the package via Arch's `makepkg` utility. After the kernel compiles, `makepkg` will prompt for an agreement to proceed with installation. Remember to return to the previous directory after the compilation is complete and the package is installed.

    [YOUR-USER-NAME@archiso aur]$ cd linux-g14
    [YOUR-USER-NAME@archiso linux-g14]$ makepkg --install --syncdeps --noconfirm --skippgpcheck
    ==> Making package: linux-g14 5.7.5.arch1-2 (Fri 26 Jun 2020 09:16:53 PM CDT)
    ==> Checking runtime dependencies...
    ==> Checking buildtime dependencies...
    ==> Retrieving sources...
      -> Cloning archlinux-linux git repo...
    Cloning into bare repository '/opt/linux-g14/archlinux-linux'...
    *omitted*
    ==> Starting build: 5.7.5-arch1-2-g14
      -> Running build hook: [base]
      -> Running build hook: [udev]
      -> Running build hook: [modconf]
      -> Running build hook: [block]
    ==> WARNING: Possibly missing firmware for module: wd719x
    ==> WARNING: Possibly missing firmware for module: aic94xx
      -> Running build hook: [encrypt]
      -> Running build hook: [filesystems]
      -> Running build hook: [keyboard]
      -> Running build hook: [fsck]
    ==> Generating module dependencies
    ==> Creating gzip-compressed initcpio image: /boot/initramfs-linux-g14-fallback.img
    ==> Image generation successful
    [YOUR-USER-NAME@archiso linux-g14]$

<!-- Separator -->

    [YOUR-USER-NAME@archiso linux-g14]$ cd ..
    [YOUR-USER-NAME@archiso aur]$

Add a boot loader entry for the patched G14-specific kernel. First, copy the existing `arch.conf` file to `arch-g14.conf`:

    [YOUR-USER-NAME@archiso aur]$ sudo cp /boot/loader/entries/{arch,arch-g14}.conf

Edit `/boot/loader/entries/arch-g14.conf` with `sudo nano` and change the `title`, `linux`, and second `initrd` lines. When done, it should look like this:

    title Arch Linux G14
    linux /vmlinuz-linux-g14
    initrd /amd-ucode.img
    initrd /initramfs-linux-g14.img
    options root=/dev/mapper/cryptroot cryptdevice=UUID=00000001-0000-4000-8000-000000000000:cryptroot rw trace_clock=local

To activate the new G14 specific kernel, `/boot/loader/loader.conf` with `sudo nano` and change the default kernel from `arch.conf` to `arch-g14.conf`. Then run `bootctl list` again to verify that the `arch-g14` entry just added will function:

    [YOUR-USER-NAME@archiso aur]$ bootctl list
    Boot Loader Entries:
            title: Arch Linux G14 (default)
               id: arch-g14.conf
           source: /boot/loader/entries/arch-g14.conf
            linux: /vmlinuz-linux-g14
           initrd: /amd-ucode.img
                   /initramfs-linux-g14.img

    Boot Loader Entries:
            title: Arch Linux
               id: arch.conf
           source: /boot/loader/entries/arch.conf
            linux: /vmlinuz-linux
           initrd: /amd-ucode.img
                   /initramfs-linux.img

    *omitted*
    [YOUR-USER-NAME@archiso aur]$

#### GPU Configuration

The NVidia drivers can now be installed. The `nvidia-dkms` package will ensure the drivers are built against the custom G14 kernel headers.

    [YOUR-USER-NAME@archiso aur]$ sudo pacman --sync --noconfirm nvidia-dkms nvidia-utils nvidia-settings nvidia-prime

The NVidia packages are distributed with a few system services that allow the NVidia driver to cooperate with laptop suspend and resume. Enable them to ensure the laptop can resume properly:

    [YOUR-USER-NAME@archiso aur]$ sudo systemctl enable nvidia-suspend.service
    Created symlink /etc/systemd/system/systemd-suspend.service.requires/nvidia-suspend.service → /usr/lib/systemd/system/nvidia-suspend.service.

<!-- Separator -->

    [YOUR-USER-NAME@archiso aur]$ sudo systemctl enable nvidia-resume.service
    Created symlink /etc/systemd/system/systemd-suspend.service.requires/nvidia-resume.service → /usr/lib/systemd/system/nvidia-resume.service.
    Created symlink /etc/systemd/system/systemd-hibernate.service.requires/nvidia-resume.service → /usr/lib/systemd/system/nvidia-resume.service.

<!-- Separator -->

    [YOUR-USER-NAME@archiso aur]$ sudo systemctl enable nvidia-hibernate.service
    Created symlink /etc/systemd/system/systemd-hibernate.service.requires/nvidia-hibernate.service → /usr/lib/systemd/system/nvidia-hibernate.service.

The following drivers are unnecessary and can cause problems if loaded. Prohibit them from loading by creating a new file at `/etc/modprobe.d/g14-prohibited-drivers.conf` with the following contents:

    blacklist nouveau
    blacklist nvidiafb
    blacklist rivafb
    blacklist i2c_nvidia_gpu

The Nvidia driver requires a few additional options to be set in order to function optimally. Create a new file at `/etc/modprobe.d/g14-nvidia.conf` with the following contents:

    options nvidia_drm modeset=1
    options nvidia NVreg_DynamicPowerManagement=0x02

X11 requires configuration. There are several different approaches available, including NVidia PRIME and reverse PRIME, outlined [here](https://lab.retarded.farm/zappel/asus-rog-zephyrus-g14/-/wikis/Hardware/Graphics). The following configuration will have the NVidia dGPU drive X11, which is undesirable for battery life, but at the present moment seems to offers the best compatibility. Create a new file at `/etc/X11/xorg.conf.d/99-g14.conf` with the following contents:

    Section "Module"
            Load "modesetting"
    EndSection

    Section "Device"
            Identifier "Nvidia Card"
            Driver "nvidia"
            VendorName "NVIDIA Corporation"
            BoardName "GeForce RTX 2060 Max-Q"
            BusID "PCI:1:0:0"
            Option "AllowEmptyInitialConfiguration"
    EndSection

#### Audio Configuration

The audio subsystem requires a configuration change in order to fix an issue with volume control. Edit the file located at `/usr/share/pulseaudio/alsa-mixer/paths/analog-output.conf.common`, and find the section titled `[Element PCM]`. Add the following text just above that section:

    [Element Master]
    switch = mute
    volume = ignore

The relevant section should look like this afterwards:

    *omitted*
    ;                                      # the PCM device index in their name, but different drivers use different
    ;                                      # numbering schemes, so we can't hardcode the full jack name in our configuration
    ;                                      # files.

    [Element Master]
    switch = mute
    volume = ignore

    [Element PCM]
    switch = mute
    volume = merge
    override-map.1 = all
    override-map.2 = all-left,all-right
    *omitted

#### Touchpad Configuration

The touchpad requires configuration in order to function properly. Create a new file at `/etc/X11/xorg.conf.d/99-g14-touchpad.conf` with the following contents:

    Section "InputClass"
            Identifier "touchpad overrides"
            MatchDriver "synaptics"
            MatchIsTouchpad "on"
            Option "TapButton1" "1"
            Option "TapButton2" "3"
            Option "VertEdgeScroll" "1"
            Option "RBCornerButton" "3"
    EndSection

#### Graphical Boot Process

First, install the `plymouth` package from the Arch User Repository (AUR).

    [YOUR-USER-NAME@archiso aur]$ git clone --depth 1 https://aur.archlinux.org/plymouth.git
    Cloning into 'plymouth'...
    remote: Enumerating objects: 23, done.
    remote: Counting objects: 100% (23/23), done.
    remote: Compressing objects: 100% (21/21), done.
    remote: Total 23 (delta 4), reused 12 (delta 2), pack-reused 0
    Unpacking objects: 100% (23/23), 26.08 KiB | 192.00 KiB/s, done.
    [YOUR-USER-NAME@archiso aur]$ cd plymouth
    [YOUR-USER-NAME@archiso plymouth]# makepkg --install --syncdeps --noconfirm
    *omitted*
    [YOUR-USER-NAME@archiso plymouth]$

<!-- Separator -->

    [YOUR-USER-NAME@archiso plymouth]$ cd ..
    [YOUR-USER-NAME@archiso aur]$

Next, install `gdm-plymouth`. The `--noconfirm` option, which prevents `makepkg` from asking the user if they wish to install the package, cannot be used with `gdm-plymouth`, as it additionally prompts the user to replace the `gdm` and `libgdm` packages already installed on the system. Instruct `makepkg` to remove `gdm` and `libgdm` so that it can replace them with equivalents that have Plymouth support.

    [YOUR-USER-NAME@archiso aur]$ git clone --depth 1 https://aur.archlinux.org/gdm-plymouth.git
    Cloning into 'gdm-plymouth'...
    remote: Enumerating objects: 8, done.
    remote: Counting objects: 100% (8/8), done.
    remote: Compressing objects: 100% (8/8), done.
    remote: Total 8 (delta 0), reused 4 (delta 0), pack-reused 0
    Unpacking objects: 100% (8/8), 4.43 KiB | 4.43 MiB/s, done.
    [YOUR-USER-NAME@archiso aur]$ cd gdm-plymouth
    [YOUR-USER-NAME@archiso gdm-plymouth]# makepkg --install --syncdeps
    *omitted*

<!-- Separator -->

    *omitted*
    autoreconf: Leaving directory `.'
    ==> Starting pkgver()...
    ==> WARNING: The package group has already been built, installing existing packages...
    ==> Installing gdm-plymouth package group with pacman -U...
    loading packages...
    resolving dependencies...
    looking for conflicting packages...
    :: gdm-plymouth and gdm are in conflict. Remove gdm? [y/N] y
    :: libgdm-plymouth and libgdm are in conflict. Remove gdm? [y/N] y

    Packages (4) gdm-3.36.2-1 [removal]  libgdm-3.36.2-1 [removal]  gdm-plymouth-3.36.2-1  libgdm-plymouth-3.36.2-1

    Total Installed Size: 5.11 MiB
    Net Upgrade Size:     0.02 MiB

    :: Proceed with installation? [Y/n] y
    (2/2) checking keys in keyring
    (2/2) checking package integrity
    *omitted*

<!-- Separator -->

    *omitted*
    Running in chroot, ignoring request: try-reload-or-restart
    (6/6) Compiling GSettings XML schema files...
    [YOUR-USER-NAME@archiso gdm-plymouth]$

<!-- Separator -->

    [YOUR-USER-NAME@archiso gdm-plymouth]$ cd ..
    [YOUR-USER-NAME@archiso aur]$

To install Plymouth onto the boot partition, `mkinitcpio` will need to be run again. First, add `plymouth` to the `HOOKS` list in `/etc/mkinitcpio.conf`, after `base` and `udev`, and replace `encrypt` with `plymouth-encrypt`:

    HOOKS=(base udev plymouth autodetect modconf block plymouth-encrypt filesystems keyboard fsck)

NOTE: At the time of this writing, the graphical boot process does not detect external monitors connected to the USB3 Type-C port. As a result, the Nvidia drivers are not configured to be available for it. It is possible this could work in the future, and if it does, the Nvidia drivers will need to be added to the `MODULES` array in `/etc/mkinitcpio.conf`.

Run `mkinitcpio -P` again, this time with `sudo`, to install Plymouth onto the boot partition.

    [YOUR-USER-NAME@archiso aur]$ sudo mkinitcpio -P

#### GNOME Display Manager (GDM)

Finally, the display manager, GDM, must be configured to boot into X11, not Wayland. Edit `/etc/gdm/custom.conf` and remove the leading comment character `#` from the line that includes `WaylandEnable=false`. Afterwards, the file should look like this:

    # GDM configuration storage

    [daemon]
    # Uncomment the line below to force the login screen to use Xorg
    WaylandEnable=false

    [security]

    *omitted*

#### Reboot into Arch

To reboot into the new Arch installation, `exit` first from your user session, then `exit` again from the `chroot` shell, and then finally run the `reboot` command:

    [YOUR-USER-NAME@archiso aur]$ exit
    logout
    [root@archiso /]# exit
    exit
    arch-chroot /mnt  28.41s user 10.18s system 1% cpu 1:03:58.08 total
    root@archiso ~ # reboot
    *omitted*

If all goes well, then the G14 should boot into a graphical login screen that will prompt for the disk decryption password, and afterwards, the user that was added earlier can sign in to GNOME.

However, if the boot fails, the installation media can still be used to correct anything that has been misconfigured. To return to the `arch-chroot` shell, the partitions must be mounted.

    root@archiso ~ # cryptsetup open /dev/nvme0n1p2 cryptroot
    Enter passphrase for /dev/nvme0n1p2: YOUR-FILESYSTEM-ENCRYPTION-PASSWORD
    cryptsetup open /dev/nvme0n1p2 cryptroot  6.25s user 0.26s system 101% cpu 6.396 total
    root@archiso ~ # mount /dev/mapper/cryptroot /mnt
    root@archiso ~ # mount /dev/nvme0n1p1 /mnt/boot
    root@archiso ~ # arch-chroot /mnt
    [root@archiso /]#

To limit the console output from the boot process to show only errors, edit `/boot/loader/entries/arch-g14.conf` once again with `sudo nano` and add `quiet splash loglevel=3 rd.systemd.show_status=auto rd.udev.log_priority=3 vt.global_cursor_default=0` to the `options`. Afterward, it should look like this:

    options quiet splash loglevel=3 rd.systemd.show_status=auto rd.udev.log_priority=3 vt.global_cursor_default=0 root=/dev/mapper/cryptroot cryptdevice=UUID=00000001-0000-4000-8000-000000000000:cryptroot rw trace_clock=local

Note: a terminal can be opened in GNOME by pressing the Windows key, typing 'terminal' in the search bar, and then pressing ENTER.

Next Steps
----------

* Visit the [asus-rog-zephyrus-g14](https://lab.retarded.farm/zappel/asus-rog-zephyrus-g14) project page
* Read through the asus-rog-zephyrus-g14 [project wiki](https://lab.retarded.farm/zappel/asus-rog-zephyrus-g14/-/wikis/home)
* Join the [Discord community](https://discord.gg/DTxNE6s)
