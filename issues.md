Issues
======

From @ZappeL:

    btw, https://gist.github.com/ntl/01cf03e327d1a6b564cc4953e501937b#gpu-configuration

    should be corrected.
    @dragonn and I found out the best way to run this is currently through iGPU (Prime offload).. 

    see: https://lab.retarded.farm/zappel/asus-rog-zephyrus-g14/-/wikis/Hardware/Graphics#igpu-as-default-main-gpu-nvidia-by-using-prime

    since 5.8 this should be used. in combination with those kernel-commandline params:
    "clocksource=tsc tsc=reliable iommu=pt"

    no other parameters, then the default ones like, dolvm, etc.

    mine for examle looks like:
    "dolvm crypt_root=UUID=d64c4cfd-69eb-4e92-aa7b-fac571c90820 root=/dev/mapper/vg0-root root_trim=yes keymap=de init=/usr/lib/systemd/systemd clocksource=tsc tsc=reliable iommu=pt"

* Reverse PRIME does not work; the next NVidia release may fix it
  - When Xorg uses the NVidia driver, fractional scaling (i.e. `xrandr --scale`) does not work
  - When Xorg uses the AMDGPU driver, the USB Type-C DisplayPort does not work
* Plymouth does not detect external display connected to USB Type-C port (via Nvidia dGPU)

Guide TODO
==========
* Add `switcheroo-control` to the packages that get installed, test dGPU on individual programs (eDP, HDMI, Type-C DP)
* Add gnome extension: `rog-core GEX`
* Add GreenWithEnvy

Other
=====
* Gnumeric uses US date format, indicates problem with LC_TIME, LC_ALL
* Hide scrolling console text when resuming from suspend
