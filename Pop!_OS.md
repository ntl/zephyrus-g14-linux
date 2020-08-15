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
