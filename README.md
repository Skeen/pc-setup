# Monitor EDID issues

Solved by modifying `/etc/default/grub` and adding:
```
drm_kms_helper.edid_firmware=edid/1920x1080.bin drm.eded_firmware=edid/1920x1080.bin
```
To `GRUB_CMDLINE_LINUX_DEFAULT`.

To effectuate this run `sudo update-grub` and reboot.

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

To compile this we want to setup `dkms`, for it:
```
sudo apt-get install dkms
sudo apt-get install linux-headers-amd64
```

Inside `src` we create the file `dkms.conf`, containing:
```
PACKAGE_NAME="r8125"
PACKAGE_VERSION="9.004.01"
BUILT_MODULE_NAME[0]="$PACKAGE_NAME"
DEST_MODULE_LOCATION[0]="/updates/dkms"
AUTOINSTALL="YES"
REMAKE_INITRD="YES"
CLEAN="rm src/@PKGNAME@.ko src/*.o || true"
```

Now we copy this `src` folder into `/usr/src/`:
```
sudo cp -R . /usr/src/r8125-9.004.01
```

And thus we are ready to build and install it with `dkms`:
```
sudo dkms add -m r8125 -v 9.004.01
sudo dkms build -m r8125 -v 9.004.01
sudo dkms install -m r8125 -v 9.004.01
```

# Booting with multiple GPUs

With multiple graphics card, and no VFIO configuration the machine will not boot.

This can be solved by modifying `/etc/default/grub` and adding:
```
iommu=soft
```
To `GRUB_CMDLINE_LINUX_DEFAULT`.

To effectuate this run `sudo update-grub` and reboot.

# Enabling VFIO

Enable VFIO by modifying `/etc/default/grub` and adding:
```
amd_iommu=on iommu=pt
```
To `GRUB_CMDLINE_LINUX_DEFAULT`.

NOTE: Replacing `iommu=soft` from the previous step.

To effectuate this run `sudo update-grub` and reboot

At this point running this:
```
#!/bin/bash
shopt -s nullglob
for g in /sys/kernel/iommu_groups/*; do
    echo "IOMMU Group ${g##*/}:"
    for d in $g/devices/*; do
        echo -e "\t$(lspci -nns ${d##*/})"
    done;
done;
```
Should return a list of IOMMU groups, confirming that IOMMU is enabled.

If the graphics card is bundled up with other devices, change the cards around physically.
