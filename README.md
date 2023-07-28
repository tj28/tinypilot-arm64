# tinypilot-arm64
A hacky solution to getting [TinyPilot](https://github.com/tiny-pilot/tinypilot) working on arm64 devices supporting USB OTG such as the Tritium H5. If you haven't bought a device yet, trust me, just get a Raspberry Pi 4.

## Overview
By manually installing the relevant .deb files and slightly modifying the install scripts, we can get this working on arm64.

## Guide for Tritium H5
1. OS: I suggest flashing [Armbian Bullseye](http://xogium.performanceservers.nl/archive/tritium-h5/archive/Armbian_23.02.2_Tritium-h5_bullseye_current_5.15.93.img.xz), as I've found using a too recent version of Debian will cause errors during runtime to the Python version being much higher than expected
2. Follow [WarheadsSE's initial guide](https://gist.github.com/WarheadsSE/f90359e0f6955478c806ecbd04a643a3). The `dts` file can be compiled via `dtc -I dts -O dtb -o h3-cc-otg.dtbo h3-cc-otg.dts`
3. Fix some dependencies since we aren't using RasPi OS, nor the original `.deb` files:
```bash
apt install -y libc6 libgcc-s1 libstdc++6 adduser python3 python3-pip python3-venv sudo nginx
touch /etc/pip.conf
chmod 775 /etc/pip.conf
echo -e "[global]\nextra-index-url=https://www.piwheels.org/simple\n" > /etc/pip.conf
```
4. Download the `ustreamer` tarball in this repo. This `.deb` was created via the `install from source` script in the TinyPilot main repo, after modifying the target architecture to be `arm64` rather than `armhf` or `arm/v7` in the source files. I then used `alien` to convert it to a tarball.
5. Extract & merge the output folders with your root directory `/`. The `install` folder does not have to be merged. I suggest using `rsync -a`
6. Run `doinst.sh` in the `install` folder. If any errors appear, remediate accordingly.
7. Repeat the same process with the `tinypilot` tarball.
8. We are now ready to run the install script. Run `curl https://raw.githubusercontent.com/tj28/tinypilot-arm64/main/get-tinypilot.sh | bash`. If all went well, there should be no errors in output. 
9. Reboot
10. I had to enable the new services manually.
```bash
systemctl enable tinypilot
systemctl enable ustreamer
systemctl enable usb-gadget
systemctl disable tinypilot-updater
```
11. Run `ls /dev` and identify which devices we are using for I/O. For me, my capture card was `/dev/video0`, and the virtual USB OTG keyboard and mouse were `/dev/hidg0` and `/dev/hidg0`, respectively.
12. Add `ustreamer_video_path: /dev/video0` to `/home/tinypilot/settings.yml` as necessary. Inspect `/home/tinypilot/app_settings.cfg` as well for KBM input.
13. Restart one more time, and if all went well, TinyPilot will be running successfully.
