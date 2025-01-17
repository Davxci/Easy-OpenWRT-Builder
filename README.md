# 🚀 Custom OpenWRT Image Build & VM Conversion Script

## A repeatable approach to custom OpenWRT image creation:
- Manage your own list(s) of OpenWRT packages
- Optionally resize default partitions to unlock extra OpenWRT filesystem storage
- Optionally convert OpenWRT images into into QEMU, Virtualbox, HyperV or VMware images
- Automated installation of "OpenWRT Imagebuilder" Linux dependencies

---

## ⚙️ Script Option Prompts

1. Enter OpenWRT STABLE release version _(or hit enter for latest snapshot)_
2. Modify partition sizes or keep OpenWRT defaults? [y/n] _(Y = follow the prompts)_
3. Enter an image filename tag _(to uniquely lablel your new image)_
4. **Optional**: Convert OWRT images to VM disk? [y/n] _(Y = select a VM format: qcow2, qed, vdi, vhdx or vmdk)_
5. **Optional**: Bake custom OpenWRT config files into the new base OWRT image _(follow on-screen directions)_
6. When the script completes, new images are located at `$(pwd)/openwrt_build_output/firmware_images`, and corresponding VM images at `$(pwd)openwrt_build_output/vm`.   
   ![image](https://github.com/itiligent/Easy-OpenWRT-Builder/blob/main/Screenshot.png)



---

## 🛠️ Prerequisites

**Any recent x86 Debian-flavored OS with the sudo package installed should be fine**. 
- All image building dependencies are automatically installed on first run.
- Windows subsystem for Linux users have additional steps. See here: https://openwrt.org/docs/guide-developer/toolchain/wsl
- If building old OpenWRT versions (< = 19.x), using a Linux distro from the same era will save you many troubles with python build machine prerequisities.

---

## 📖 Instructions

1. 📥 Download the image builder script and make it executable:
   ```
   chmod +x x86-imagebuilder.sh
   ```

2. 🛠️ Customize your package recipie in the `CUSTOM_PACKAGES` section at the top of the script (the included list of packages are examples and can be changed). If you have issues with building, check builder output for package conflicts. (OpenWRT's new **apk** package manager will hopefully help alleviate package conflicts).


3. ▶️ Run the script **without** sudo (it will prompt for sudo) and follow the prompts:
   ```
   ./x86-imagebuilder.sh
   ```
4. VMware ESXi users see [here](https://github.com/itiligent/Easy-OpenWRT-Builder/blob/main/OWRT-ON-ESXi.md) for extra required steps.

---

## 📂 Persistent filesystem expansion WITHOUT resizing partitions

It is possible to add a large **persistent** EXT4 data partition that, unike any resized partitions, won't be wiped by a reset or sysupgrade.

Add a usb drive (or vdisk) with an EXT4 partition and mount this with:
   ```
   block detect | uci import fstab
   uci set fstab.@mount[-1].enabled='1'
   uci commit fstab
   reboot
   ```
Alternately, see http://routername/cgi-bin/luci/admin/system/mounts to mount the new disk via Luci
   
2. Copy the (now updated) OpenWRT `/etc/fstab file`
4. Re-run the build script, this time adding the copied `/etc/fstab` file to `$(pwd)/openwrt_inject_files` when prompted.
   - The updated fstab file that references the newly mounted EXT4 partition will now be baked into the new build, making this extra storage persistent after a reset or sysupgrade.
   -  Note that **OWRT Image Builder does not support sysupgrade for on x86 images**. Instead, for x86 upgrades simply overwite your virtual machine OS disk with a new image.

---

# ⚠️ WARNING

- **Parition resize should only be used with x86 builds**.
  - Resize of firmware paritions with non x86 (i.e. router NAND flash memory) **will 99.99999% brick your device!!**
- **If you are an OpenWRT Jedi and you do modify partition sizes on anything other than x86, you will no longer be able to use sysupgrade**.
  - Running sysupgrade images over a build with resized partitions will return the default partition schema, which will likely be a bad thing. Instead follow the same upgrade method for x86 above.

