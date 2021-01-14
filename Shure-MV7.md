# Shure MV7 Recipes

## Configuration

Open configuration files in editor:

```
sudo vim /etc/pulse/daemon.conf /lib/udev/rules.d/91-shure-mv7.rules /usr/share/pulseaudio/alsa-mixer/profile-sets/shure-mv7.conf
```

Edit `/etc/pulse/daemon.conf`, and find the line `; default-sample-rate = 44100`. Add:

```
; default-sample-rate = 44100
default-sample-rate = 48000
; alternate-sample-rate = 48000
```

Create `/lib/udev/rules.d/91-shure-mv7.rules`:

```
SUBSYSTEM!="sound", GOTO="pulseaudio_end"
ACTION!="change", GOTO="pulseaudio_end"
KERNEL!="card*", GOTO="pulseaudio_end"

SUBSYSTEMS=="usb", ATTRS{idVendor}=="14ed", ATTRS{idProduct}=="1012", ENV{PULSE_PROFILE_SET}="shure-mv7.conf"

LABEL="pulsaudio_end"
```

Create `/usr/share/pulseaudio/alsa-mixer/profile-sets/shure-mv7.conf`:

```
; Shure MV7
;
; See default.conf for an explanation on the directives used here.

[General]
auto-profiles = no

[Mapping analog-stereo-headphone]
description = Headphone
device-strings = hw:%f
channel-map = front-left,front-right
direction = output

[Mapping analog-input-mic]
description = Microphone
device-strings = hw:%f
channel-map = mono
direction = input

[Profile output:analog-stereo-headphone+input:analog-input-mic]
description = Microphone+Headphone
output-mappings = analog-stereo-headphone
input-mappings = analog-input-mic
skip-probe = yes
```

Reload udev:

```
sudo udevadm control --reload-rules
sudo udevadm trigger
```

Restart Pulse audio:

```
systemctl --user restart pulseaudio.socket
```
