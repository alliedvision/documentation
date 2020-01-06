# Additional documentation for ARM boards with i.MX6
* Changing driver parameters with devicetree
* Changing memory size reserved for CMA
* Enabling and disabling MIPI CSI-2 camera drivers

## Changing driver parameters with devicetree
### Purpose of devicetree
Some use cases require changing camera driver parameters such as the MIPI clock or 
the number of lanes. 
To achieve this, devicetree files are used. The underlying principles are valid for all boards. 
### Devicetree introduction
Devicetree source files (DTS, file ending .dts) and devicetree source include files (.dtsi) describe a device as 
a human-readable tree of nodes and their properties. The DTS file is compiled into a binary devicetree blob file (DTB, file ending .dtb) that the bootloader puts into RAM. During the boot process, the kernel takes over the values from the devicetree. 
### Modifying and compiling the Allied Vision camera devicetree
You can modify the Allied Vision camera devicetree directly on the embedded board:
1. Install the DTC (devicetree compiler) on the board: 
```console
# Install the DTC
sudo apt-get install device-tree-compiler
```
2. Identify the correct devicetree blob for your board. Examples: 
* Nitrogen6_MAX: /boot/imx6qp-nitrogen6_max.dtb. 
* Wandboard: /media/ubuntu/<boot partition>/imx6qp-wandboard-revd1.dtb
3. Decompile the devicetree blob to devicetree source. In this example, we use Nitrogen6_MAX. Ignore warnings during this procedure: 
```console
# Decompile the devicetree blob
./dtc -I dtb -O dts -o ./devicetree.dts /boot/imx6qp-nitrogen6_max.dtb
```
4. Make the desired changes to the devicetree source file devicetree.dts:
```console
avt_imx6_csi2: avt_imx6_csi2@3C 
{
    compatible = "avti,avt_imx6_csi2";
    reg = <0x3C>;                      
# I2C camera address
    clocks = <&clks IMX6QDL_CLK_PWM3>; // Pixel clock frequency
    clock-names = "csi_mclk";
    av_cam_i2c_clk = <400000>;         
# Host I2C camera clock
# i.MX6 has two IPUs (IPU0 and IPU1), each IPU has CSI channels CSI0 and CSI1. 
# The virtual channel is defined based on the IPU and CSI selection. 
# Default: virtual channel 0. 
    ipu_id = <0>;          
# IPU ID value of the connected camera to choose stream and VC
    csi_id = <0>;          
# CSI ID value of the connected camera to choose stream and VC
    lanes = <4>;           
# Lane count. No effect when lanes_auto_conf = 1 
    lanes_auto_conf = <1>; 
# Value 1 sets lane auto negotiation and configuration. 
    mclk = <188000000>;    
# Master clock
    mclk_source = <0>;
    clk_auto_conf = <1>;   
# Value 1 sets clock auto negotiation and configuration
};
```
5. Compile the devicetree source to devicetree blob and replace the original one. In this example, we use Nitrogen6_MAX. Ignore warnings during this procedure: 
```console
# Decompile the devicetree blob
sudo ./dtc -I dts -O dtb -o /boot/imx6qp-nitrogen6_max.dtb ./devicetree.dts
```
6. Reboot the board to apply the changes.

# Changing memory size reserved for CMA
## Introduction
The Linux kernel automatically allocates a part of its main memory reserved for CMA (Contiguous Memory Allocator). Increasing the memory size reserved for CMA automatically decreases the available user space RAM and vice versa. CMA is used by the DMA API.
Change the default value of memory size reserved for CMA to:
* Reach a higher frame rate by using more buffers (if the current value limits it)
* Increase performance when capturing images with more than 5 megapixels
* Increase available user space RAM for image processing applications (if the memory size reserved for CMA can be reduced for your use case)

When you change the CMA value, be aware that it is also used by other modules than the camera driver, for example, the GPU driver. 
## Modifying the devicetree file
As a first step, modify the devicetree file, for example, arch/arm/boot/dts/imx6q.dtsi. 

```
// .dtsi file modification to increase CMA reserved memory to approx. 800 MB

--- a/arch/arm/boot/dts/imx6q.dtsi
+++ b/arch/arm/boot/dts/imx6q.dtsi
@@ -93,7 +93,7 @@
         linux,cma {
             compatible = "shared-dma-pool";
             reusable;
-            size = <0x14000000>;
+            size = <0x30000000>;
             linux,cma-default;
         };
```
Additionally, modify the value of CONFIG_CMA_SIZE_MBYTES to, for example, 800:
1. Go to the arch/arm/configs/<boardname>_alliedvision_defconfig file. By default, the CMA size value is 320 MBytes:
`CONFIG_CMA_SIZE_MBYTES=320`
2. Change the value.
3. Rebuild the kernel. To do this, use our install and build scripts.

To apply temporary changes, if you want to try a value:
You can change the kernel behavior with menuconfig (the kernel configuration menu). For temporary changes, rebooting is sufficient. Applying changes permanently requires you to recompile the kernel. 

```console
# If not already present on your system, install libncurses-dev
sudo apt-get libncurses-dev

# Make menuconfig
make menuconfig    
```
In the kernel configuration menu:
1. Select Device Drivers > Generic Driver Options and set * for DMA Contiguous Memory Allocator.
2. Select Size in Mega Bytes, press the Return key, and enter the desired value. 
3. Save.
4. Select Exit until the kernel configuration menu closes.
5. Reboot.

Now the new CMA value is applied temporarily.
To apply the new CMA value permanently, rebuild the kernel.
## Tested values
### Wandboard and Nitrogen6_MAX
We have tested these values for the CMA size:

CMA value | Resolution | Maximum buffers
------------ | ------------- | -------------
1200 | 5632×4096 | 14
1000 | 5632×4096 | 10
800 | 5632×4096 | 8
640 | 5632×4096 | 5

# Enabling and disabling MIPI CSI-2 camera drivers
By default, the camera driver modules of MIPI CSI-2 cameras from Allied Vision and Omnivision are loaded. 
You can enable or disable driver modules either permanently or until the next bootup.

Note that Allied Vision’s driver avt_imx6_csi2 is built on NXP’s driver mxc_v4l2_capture. 
Therefore, mxc_v4l2_capture must be enabled to use Alvium CSI-2 cameras.

To disable or enable a MIPI CSI-2 driver module until the next bootup:

```shell
# List all loadable modules 
lsmod

# Disable modules
sudo rmmod ov5642_camera 

# Depends on mxc_v4l2_capture, v4l2_int_device
sudo rmmod avt_imx6_csi2 

# Depends on mxc_v4l2_capture, v4l2_int_device
# Keep enabled to use your camera
sudo rmmod mxc_v4l2_capture 
sudo rmmod v4l2_int_device

# Enable a module
sudo modprobe avt_imx6_csi2
```

To disable the Omnivision driver module permanently, go to /etc/modprobe.d/blacklist.conf 
and append a line with the driver module you want to disable: 

```shell
#  Go to the end of the .conf file. Append a line with the driver module you want to disable
blacklist ov5642_camera
```
