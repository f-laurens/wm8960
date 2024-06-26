# Tagtagtag sound card driver for RaspberryPi

Sound card driver for Nabaztag and Nabaztag:tag Raspberry Pi HAT "Tagtagtag" 2019 card, v4.3 featuring:

- a WM8960 codec driving Nabaztag's 8Ω mono speaker on right output
- a MAX9759ETE+T amplifier plugged on WM8960's headphone output (merging left and right) and driving Nabaztag:tag's 4Ω speaker, controlled by GPIOs 7 (/SHDN) and 8 (/MUTE)
- a line out plug detect on GPIO 25 (Nabaztag:tag only, and if user actually plugged line out)
- a physical volume button on GPIO 22 and 27.

WM8960 codec is based from linux codec from d2912cb15bdda8ba4a5dd73396ad62641af2f520 commit of [wm8960.c](https://github.com/torvalds/linux/blob/d2912cb15bdda8ba4a5dd73396ad62641af2f520/sound/soc/codecs/wm8960.c) and [wm8960.h](https://github.com/torvalds/linux/blob/d2912cb15bdda8ba4a5dd73396ad62641af2f520/sound/soc/codecs/wm8960.h) files.

Compared to upstream, the codec driver configures clocking in slave mode and made compatible with simple-audio-card.

MAX9759 codec has been patched to make gain GPIOs optional.

Volume button is handled by an additional driver exposing GPIOs as ALSA switches.
Line-out and volume button are handled by a separate daemon that modifies sound volumes.

## Datasheets

- WM8960: https://statics.cirrus.com/pubs/proDatasheet/WM8960_v4.4.pdf
- MAX9759ETE+T: https://datasheets.maximintegrated.com/en/ds/MAX9759.pdf

Sound card schematics will be published on https://github.com/nabaztag2018/hardware

## Installation

Install requirements

    sudo apt-get install raspberrypi-kernel-headers libasound2-dev

Clone source code.

    git clone -b tagtagtag-sound https://github.com/pguyot/wm8960

Compile and install with

    cd wm8960
    make
    sudo make install

Makefile will automatically edit /boot/config.txt and add/enable if required the following params and overlays:

    dtparam=i2c_arm=on
    dtoverlay=i2s-mmap
    dtparam=i2s=on
    dtoverlay=tagtagtag-sound

Makefile will also reset card's ALSA Mixer settings to defaut values.

You might want to review changes before rebooting.

Reboot.

Mixer daemon uses configuration file `/var/lib/tagtagtag-sound/mixer.conf`.
Any change to this file is taken into account by sending a USR1 signal to the daemon.
The pid is saved to `/run/tagtagtag-mixerd.pid`
