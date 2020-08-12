# RescueOS

RescueOS is a rescue solution for the Nokia N900, distributed as an `initrd`
image.  It has several features, most notably:

- Mounting Maemo `/`
- EMMC access (can mount Maemo home and MyDocs partitions)
- Lock code reset
- WiFi support
- USB mass-storage mode
- USB networking
- Battery charging

Users must be familiar with the Linux console.

This project is *not* the Meego rescue initrd.

# How it works

The flasher utility loads RescueOS.  This makes it unnecessary to modify the
bootloader or Maemo's `/sbin/preinit`, unlike other solutions.

By using the `-l` option, we don't flash the kernel nor the Maemo `initrd`
image.  The kernel is only loaded into RAM.  No modification to the NAND nor
bootloader happens.

# Use cases

Most users don't need it.  That said, this can be used to:

- Fix typos in bootscripts which prevent the OS (Maemo) from booting
- Creating a backup of files before starting a reflash
- Charging the battery (when Maemo is dead)
- Modification of the EMMC partitions and partition table
- Backing up and restoring Maemo rootfs
- `fsck` file system checks
- Resetting the lock code (without reflashing)

# Booting RescueOS

Note: You can't store persistent data in /, since it'ss an initrd.  If you mount
the EMMC or the SD card, they can be written to normally.

**Note**: This comes without any promises and without any warranty.  Therefore,
you're doing everything at your own risk.

To load RescueOS onto an N900, you'll need a USB data cable, a PC, and one of
the following flashers:

- 0xFFFF: the Open Free Fiasco Firmware Flasher
  - Free/open source, but only runs on GNU/Linux
  - Packaged for Ubuntu and Debian as `0xffff`:
    - [Ubuntu](http://packages.ubuntu.com/source/0xffff)
    - [Debian](https://packages.debian.org/sid/0xffff)
  - [Discussion](http://talk.maemo.org/showthread.php?t=87996)
  - [Source](https://github.com/pali/0xFFFF)
- `maemo_flasher` v3.5: the original proprietary flasher from Nokia, for
  Windows, MacOS, and GNU/Linux, but the official distribution site is gone:
  - [Archive](https://web.archive.org/web/20131117084237/http://skeiron.org/tablets-dev/maemo_dev_env_downloads/)
  - [Documentation](http://wiki.maemo.org/Documentation/Maemo_5_Developer_Guide/Development_Environment/Maemo_Flasher-3.5)

Switch the N900 completely off.  If it's plugged into a charger, a PC, or
anything else via the USB port, unplug it and wait about 10 seconds to give the
charger software time to shut down.  Don't connect it to the PC yet.

Start the flasher and get it ready to load RescueOS.  For `0xFFFF`, the command
is:

    0xFFFF -m kernel:rescueOS_n900_kernel_1.3.zImage \
      -m initfs:rescueOS-1.3.img \
      -l -b 'rootdelay root=/dev/ram0'

For `maemo_flasher`, it's:

    flasher-3.5 -k rescueOS_n900_kernel_1.3.zImage \
      -n rescueOS-1.3.img \
      -l -b"rootdelay root=/dev/ram0"

You'll get a message like "suitable device not found..." but the flasher should
block and wait.  Now connect the N900 to the PC with the data cable; the flasher
should detect the device, upload RescueOS, launch it, and then exit.

# How to use it

It boots and you get a bash shell.

The folder `/rescueOS` contains some scripts for various tasks.  Not all of them
are completed yet.  They help you with the usual stuff, e.g. mass-storage
mode, Maemo root mounting, USB networking, WiFi setup, etc.

# Mounting Maemo root

`/rescueOS/mount-maemo-root.sh` mounts the Maemo root to `/mnt/maemo`.
`/rescueOS/umount-maemo-root.sh` unmounts it.

Lock code reset

`code_reset` will set the lock code back to the default (`12345`).

# USB networking

`/rescueOS/usbnetworking-enable.sh` sets up USB networking:

```shell
ifconfig usb0 192.168.2.15 up
ifconfig usb0 netmask 255.255.255.0
route add 192.168.2.14 usb0
```

Disable it with `/rescueOS/usbnetworking-disable.sh`.

# USB mass-storage mode

`/rescueOS/mass-storage-enable.sh` makes the first two EMMC partitions available
for mass-storage mode.  These are respectively `/home` and `/home/user/MyDocs`
inside Maemo.

`/rescueOS/mass-storage-disable.sh` deactivates mass-storage mode.  Remember to
unmount the device on your PC first.

# WiFi support

RescueOS has `wpa_supplicant`, but *without* EAP support.  This should be fine
for most home networks which use a PSK.

If you use a passphrase instead of a passkey, try using:

```shell
wpa_passphrase $ESSID $PASSPHRASE > /run/wlan.conf
sh /rescueOS/setup-wpa-wifi.sh
```

Replace `$ESSID` and `$PASSPHRASE` as appropriate.

For DHCP, use `udhcpc -i wlan0`.

# Battery charging

`/rescueOS/charge21.bash` is based upon ShadowJK's
[charge21](http://enivax.net/jk/n900/) script.  Use it only with a wall charger,
as it pulls 950 mA, and USB isn't designed for that.  If you're sure your USB
port can push an amp and you can live with the consequences of burning out your
USB bus if you're wrong, hey, it's your computer.

# Shutdown

`poweroff`

# Root password

`rootme`

# Shortcuts

The following shortcuts are available to reduce typing on the N900 keyboard.

- `mmr`: Mount Maemo root
- `ummr`: Unmount Maemo root
- `enftp`: Enable FTP (anonymous upload and download everywhere)
- `mse`: Enable mass-storage mode
- `msd`: Disable mass-storage mode
- `usbne`: Enable usb networking
- `usbnd`: Disable usb networking
- `chargebat`: Start battery charging
- `telnetd`: Start `telnetd` (only necessary if you've stopped it explicitly,
  since it starts on boot)

In addition, <kbd>Tab</kbd> can be generated with <kbd>Shift</kbd> + <kbd>Space</kbd> and 
<kbd>Esc</kbd> can be generated with <kbd>Fn</kbd> + <kbd>Ctrl</kbd> (the
<kbd>Fn</kbd> key is marked with <kbd>&#x2197;</kbd>.

# File transfer

Enable the FTP server with `enftp`.  It will have anonymous upload and download
enabled everywhere.  You can connect to it via USB networking or over WiFi,
using any FTP client.

Alternatively, use mass-storage mode.

# Afterword

Contributions and suggestions are welcome.  Write to the original author (NIN101
on freenode or TMO, or `n900-rescueos` at `quitesimple` period `org`).

This fork integrates most of the changes from `gnoutchd` branch of the original,
and includes a change made to the [HACKING](HACKING.md) document made by @gnoutchd
which describes how to verify signatures properly.  Feel free to open an issue
or pull request here if you have any questions.
