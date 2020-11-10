# NVIDIA Jetson driver 

## Installation instructions
Installation instructions are provided in the README of the driver repository:   
https://github.com/alliedvision/linux_nvidia_jetson

## Support of two cameras
You can connect and operate two Alvium cameras with CSI-2 interface.

## Restrictions

### Cropping
 Alvium CSI-2 cameras with firmware version 0.1.0 don't support cropping of the width.
 
 ### Image widths not supported by NVIDIA
NVIDIA's video input has restrictions concerning the image width depending on the pixel format. If the camera sends an image width not divisible by 16, 32, or 64 (according to the current pixel format), there is a padding at the end of each line. Some tools such as qv4l2 don't support padding bytes, so that these images are displayed incorrectly.

**Workaround:** 
Enable cropping (see BasicV4L2 in the examples repository) and use a supported value.

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
