# NVIDIA Jetson driver 

## Installation instructions
Installation instructions are provided in the README of the driver repository:   
https://github.com/alliedvision/linux_nvidia_jetson

## Additional information

### Support of two cameras
You can connect and operate two Alvium cameras with CSI-2 interface.

### Restrictions of certain image widths, dependent on camera sensor and pixel format
* Alvium CSI-2 cameras with firmware version 0.1.0 don't support cropping of the width.
* NVIDIA's video input has restrictions concerning the image width depending on the pixel format. 
If the camera sends an image width not divisible by 16, 32, or 64 (according to the current pixel format), there will be a padding at the end of each line.
In that case, some tools or libraries such as qv4L2 do not display images correctly.
Please see the list with supported and unsupported pixel formats:   
https://www.alliedvision.com/fileadmin/content/images/Software/Jetson-restrictions.PNG

### Supported number of lanes
Currently, the driver supports using 4 lanes.

### Special features, troubleshooting
The V4L2 node provides some features that are no camera features, but features of the NVIDIA video input unit (VI).
It is not recommended to change these features because the stream might fail or image data gets corrupted.
Rebooting Jetson TX2 resets these features.
* frame_timeout  
* height_align 
* write_isp_format 
* sensor_modes 
* disable_frame_timeout 
* disable_stride_aligment 
* low_latency_mode
