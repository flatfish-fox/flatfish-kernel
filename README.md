Dear developers,

Welcome to the Mozilla Firefox OS Contribution Program for Flatfish. For those who wish to modify the kernel, you will learn in this concise guide how to:

* **Download the toolchain and get sources**
* **Download the necessary patches**
* **Build the kernel**

Before compiling the Flatfish kernel, please first download and compile the B2G source codes for Flatfish. For more information, please visit the **["MozillaWiki"]** website, you will have the **"b2g_flatfish"** folder on your **$WORKSPACE**

    export WORKSPACE=$YOUR_B2G_WORKSPACE

The following steps will allow you to compile and update your Flatfish boot image (Ubuntu 12.04 x64):

1. Acquire the Allwinner A31 kernel source
 * Visit the website **http://linux-sunxi.org/A31**
 * Find the **["External links"]** and get into **["Allwinner A31 kernel source code"]** repository 
 * Find the revision tag: **["A31-Android4.2-v3.2"]** and get the snapshot, extract the tarball, move and rename the linux source folder to **$WORKSPACE/linux**
 
2. Download Flatfish kernel patch and ramdisk.img
 ```
 cd $WORKSPACE
 git clone https://github.com/flatfish-fox/flatfish-kernel.git 
 ```

3. Patch the kernel source
 ```
 cd $WORKSPACE/linux
 patch -p1 < $WORKSPACE/flatfish-kernel/0001-Allwinner-A31-kernel-patches-for-flatfish-tablet.patch
 ```

4. Download and set the cross compiler path for compiling the kernel
 ```
 cd $WORKSPACE
 wget https://launchpad.net/linaro-toolchain-binaries/trunk/2012.02/+download/gcc-linaro-arm-linux-gnueabi-2012.02-20120222_linux.tar.bz2
 tar jxvf gcc-linaro-arm-linux-gnueabi-2012.02-20120222_linux.tar.bz2
 export PATH=$WORKSPACE/gcc-linaro-arm-linux-gnueabi-2012.02-20120222_linux/bin:$PATH
 ```

5. Compile the kernel
 ```
 cd $WORKSPACE/linux
 ./build.sh -p sun6i_fiber 
 ```

6. Generate the boot.img 
 ```
 mkdir $WORKSPACE/bootimage
 cd $WORKSPACE/bootimage
 cp -a $WORKSPACE/flatfish-kernel/ramdisk.img ramdisk.img
 cp -a $WORKSPACE/linux/bImage kernel
 $WORKSPACE/b2g_flatfish/out/host/linux-x86/bin/mkbootimg --kernel kernel --ramdisk ramdisk.img --cmdline "console=ttyS0,115200 rw init=/init androidboot.hardware=flatfish ion_reserve=256M loglevel=7" --base 0x40000000 --output boot.img
 ```

7. Replace kernel modules 
 ```
 cp -a $WORKSPACE/linux/output/lib/modules/3.3.0/* $WORKSPACE/b2g_flatfish/backup-flatfish/system/vendor/modules
 ```

8. Recompile the B2G
 * For this part, please follow the instructions at **["MozillaWiki"]** website

9. Update the boot.img
 * Connect your Flatfish tablet device via USB.
 ```
 sudo adb reboot boot-fastboot
 sudo fastboot flash boot $WORKSPACE/bootimage/boot.img
 sudo fastboot flash system $WORKSPACE/b2g_flatfish/out/target/product/flatfish/system.img
 sudo fastboot reboot
 ```
["MozillaWiki"]:https://wiki.mozilla.org/FirefoxOS/TCP/Patching
["External links"]:http://linux-sunxi.org/A31#External_links
["Allwinner A31 kernel source code"]:http://git.rhombus-tech.net/linux
["A31-Android4.2-v3.2"]:http://git.rhombus-tech.net/?p=linux.git;a=commit;h=1636bffddc2832b4574a80324e362742fb2ecd7d
