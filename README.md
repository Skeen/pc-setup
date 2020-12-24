# Monitor EDID issues

Solved by modifying `/etc/default/grub` and adding:
```
drm_kms_helper.edid_firmware=edid/1920x1080.bin drm.eded_firmware=edid/1920x1080.bin
```
To `GRUB_CMDLINE_LINUX_DEFAULT`.

# Missing Ethernet support

Solved by downloading driver source code from:
* https://www.realtek.com/en/component/zoo/category/network-interface-controllers-10-100-1000m-gigabit-ethernet-pci-express-software

The driver: "2.5G Ethernet LINUX driver r8125 for kernel up to 5.6".

The download contains a `.tar.bz2` file which can be unpacked, inside is:
```
autorun.sh
Makefile
README
src
```

Running `sudo bash autorun.sh` will blacklist the old realtek driver, and build + install the new one.
