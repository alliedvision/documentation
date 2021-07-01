====================
NVIDIA Jetson driver 
====================

Installation instructions
=========================

Installation instructions are provided in the README of the driver repository:   
https://github.com/alliedvision/linux_nvidia_jetson

Changes
=======

Changes in driver version 2.0.0
-------------------------------
Exposure active signals
^^^^^^^^^^^^^^^^^^^^^^^^
The Jetson driver now supports exposure active signals. Note that the camera requires firmware version 00.04.00.34658 or higher. For information about 
exposure active, see the Alvium MIPI CSI-2 Cameras Direct Register Access Controls Reference:   
https://www.alliedvision.com/en/support/technical-documentation/alvium-csi-2-documentation.html   

Note especially that exposure active output line mode must be set to Off before output lines can be changed.   

Min/max values for exposure and gain
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Min/max values for exposure auto and auto gain are available. Note that the values are limited to 32 bits.

Support of two cameras
=======================

You can connect and operate two Alvium cameras with CSI-2 interface.

Restrictions
============

Cropping
--------

Alvium CSI-2 cameras with firmware version 0.1.0 don't support cropping of the width.
 
Image widths not supported by NVIDIA
------------------------------------

NVIDIA's video input has restrictions concerning the image width depending on the pixel format. If the camera sends an image width not divisible by 16, 32, or 64 (according to the current pixel format), there is a padding at the end of each line. Some tools such as qv4l2 don't support padding bytes, so that these images are displayed incorrectly.

**Workaround:** 
Enable cropping (see BasicV4L2 in the examples repository) and use a supported value.

Supported number of lanes
-------------------------

Currently, the driver supports using 4 lanes.

Special features, troubleshooting
----------------------------------

The V4L2 node provides some features that are no camera features, but features of the NVIDIA video input unit (VI).
It is not recommended to change these features because the stream might fail or image data gets corrupted.
Rebooting Jetson TX2 resets these features.

-  frame_timeout  
-  height_align 
-  write_isp_format 
-  sensor_modes 
-  disable_frame_timeout 
-  disable_stride_aligment 
-  low_latency_mode

Supported pixel formats
=======================

The following tables lists supported pixel formats on Jetson boards of the current driver. 
Previous driver versions don't support 10-bit and 12-bit RAW formats.
On Jetson TX2 and Jetson Xavier, 10-bit and 12-bit RAW formats deviate 
from the V4L2 standard  and custom formats used. Compared to the standard formats, 
the custom format contains "TX2" or "XAVIER". The custom 4CC formats contain "Y" for Xavier and "2" for TX2.


+----------+-------------------------------------+
| Platform | Pixel format name                   |
+==========+=====================================+
| Standard | V4L2_PIX_FMT_Y10 ('Y10')            |
+----------+-------------------------------------+
| TX2      | V4L2_PIX_FMT_TX2_Y10 ('J2Y0')       |
+----------+-------------------------------------+
| Xavier   | V4L2_PIX_FMT_XAVIER_Y10 ('JXY0')    |
+----------+-------------------------------------+

Compared to the standard formats, 
the bits on Jetson TX2 and Xavier are shifted to the left and there are one or two zero bits at the beginning. 
In the example below, there are two zero bits and only B9 to B0 contain productive data. The low-order bits are 
undefined and therefore listed as "X". 
 
10-bit and 12-bit formats are padded to 16 bits per channel.

V4L2_PIX_FMT_Y10 bit order in a 16-bit word:

0 0 B9 B8 B7 B6 B5 B4 B3 B2 B1 B0 X X X X


The Byte order is always identical with the standard pixel format. 

Standard formats and byte layouts are documented at: 

https://www.kernel.org/doc/html/v4.15/media/uapi/v4l/pixfmt-rgb.html

https://www.kernel.org/doc/html/v4.15/media/uapi/v4l/yuv-formats.html



Nano
----

On Nano, all supported pixel formats comply with the V4L2 standards.

**Monochrome**

+-------+--------+-------------------+
| Depth | FourCC | Enumerator        |
|       |        |                   |
+=======+========+===================+
| 8     | GREY   | V4L2_PIX_FMT_GREY |
+-------+--------+-------------------+
| 10    | Y10    | V4L2_PIX_FMT_Y10  |
+-------+--------+-------------------+
| 12    | Y12    | V4L2_PIX_FMT_Y12  |
+-------+--------+-------------------+

**Bayer**

+----------+-------+--------+------------------------+
|Pattern   | Depth | FourCC | Enumerator             |
+==========+=======+========+========================+
|RGRG, GBGB| 8     | RGGB   | V4L2_PIX_FMT_SRGGB8    |
|          +-------+--------+------------------------+
|          | 10    | RG10   | V4L2_PIX_FMT_SRGGB10   |
|          +-------+--------+------------------------+
|          | 12    | RG12   | V4L2_PIX_FMT_SRGGB12   |
+----------+-------+--------+------------------------+
|GRGR, BGBG| 8     | GRBG   | V4L2_PIX_FMT_SGRBG8    |
|          +-------+--------+------------------------+
|          | 10    | BA10   | V4L2_PIX_FMT_SGRBG10   |
|          +-------+--------+------------------------+
|          | 12    | BA12   | V4L2_PIX_FMT_SGRBG12   |
+----------+-------+--------+------------------------+
|GBGB, RGRG| 8     | GBRG   | V4L2_PIX_FMT_SGBRG8    |
|          +-------+--------+------------------------+
|          | 10    | GB10   | V4L2_PIX_FMT_SGBRG10   |
|          +-------+--------+------------------------+
|          | 12    | GB12   | V4L2_PIX_FMT_SGBRG12   |
+----------+-------+--------+------------------------+
|BGBG, GRGR| 8     | BGGR   | V4L2_PIX_FMT_SBGGR8    |
|          +-------+--------+------------------------+
|          | 10    | BG10   | V4L2_PIX_FMT_SBGGR10   |
|          +-------+--------+------------------------+
|          | 12    | BG12   | V4L2_PIX_FMT_SBGGR12   |
+----------+-------+--------+------------------------+

**RGB**

RGB3 is unsupported. X bytes are set to 0 by NVIDIA's video input unit.

+----------+-------+--------+------------------------+---------------+
|Layout    | Depth | FourCC | Enumerator             | Memory layout |
|          |       |        |                        +---------------+
|          |       |        |                        | Byte 0-1-2-3  |
+==========+=======+========+========================+===============+
| XRGB     | 8     | BX24   | V4L2_PIX_FMT_XRGB32    | B-G-R-X       |
+----------+-------+--------+------------------------+---------------+
| BGRX     | 8     | XR24   | V4L2_PIX_FMT_XBGR32    | X-R-G-B       |
+----------+-------+--------+------------------------+---------------+

**YUV**

The camera outputs this format as UYVY instead of VYUY.

+----------+-------+--------+------------------------+---------------+
|Layout    | Depth | FourCC | Enumerator             | Memory layout |
|          |       |        |                        +---------------+
|          |       |        |                        | Byte 0-1-2-3  |
+==========+=======+========+========================+===============+
| XRGB     | 8     | VYUY   | V4L2_PIX_FMT_VYUY      | Cr0-Y0-Cb0-Y1 |
+----------+-------+--------+------------------------+---------------+

Xavier AGX and NX
-----------------

Bit order of 10-bit and 12-bit custom formats on Xavier AGX and NX:

+-------+---------------------------------------------------+
| Depth | Bit order (X is undefined)                        |  
+=======+===================================================+
| 10    |0 B9 B8 B7 B6 B5 B4 B3 B2 B1 B0 X X X X X          | 
+-------+---------------------------------------------------+
| 12    |0 B11 B10 B9 B8 B7 B6 B5 B4 B3 B2 B1 B0 X X X      |
+-------+---------------------------------------------------+

**Monochrome**

Monochrome standard format:

+-------+--------------+----------------------+
| Depth | FourCC       | Enumerator           |
+=======+==============+======================+
| 8     | GREY         | V4L2_PIX_FMT_GREY    |
+-------+--------------+----------------------+

Monochrome custom formats:

+-------+--------------+-----------------------------+
| Depth | FourCC       | Enumerator                  | 
+=======+==============+=============================+
| 10    | JXY0         | V4L2_PIX_FMT_XAVIER_Y10     | 
+-------+--------------+-----------------------------+
| 12    | J2Y2         | V4L2_PIX_FMT_XAVIER_Y12     |
+-------+--------------+-----------------------------+

**RGB**

RGB3 is unsupported. X bytes are set to 0 by NVIDIA's video input unit.

+----------+-------+--------+------------------------+---------------+
|Layout    | Depth | FourCC | Enumerator             | Memory layout |
|          |       |        |                        +---------------+
|          |       |        |                        | Byte 0-1-2-3  |
+==========+=======+========+========================+===============+
| XRGB     | 8     | BX24   | V4L2_PIX_FMT_XRGB32    | B-G-R-X       |
+----------+-------+--------+------------------------+---------------+
| BGRX     | 8     | XR24   | V4L2_PIX_FMT_XBGR32    | X-R-G-B       |
+----------+-------+--------+------------------------+---------------+

**Bayer**

8-bit standard formats:

+----------+-------+--------+----------------------------+
|Pattern   | Depth | FourCC | Enumerator                 |
+==========+=======+========+============================+
|RGRG, GBGB| 8     | RGGB   | V4L2_PIX_FMT_SRGGB8        |
+----------+-------+--------+----------------------------+
|GRGR, BGBG| 8     | GRBG   | V4L2_PIX_FMT_SGRBG8        |
+----------+-------+--------+----------------------------+
|GBGB, RGRG| 8     | GBRG   | V4L2_PIX_FMT_SGBRG8        |
+----------+-------+--------+----------------------------+
|BGBG, GRGR| 8     | BGGR   | V4L2_PIX_FMT_SBGGR8        |
+----------+-------+--------+----------------------------+


10-bit and 12-bit custom formats:

+----------+-------+--------+----------------------------+
|Pattern   | Depth | FourCC | Enumerator                 |
+==========+=======+========+============================+
|RGRG, GBGB| 10    | JXR0   | V4L2_PIX_FMT_XAVIER_SRGGB10|
|          +-------+--------+----------------------------+
|          | 12    | JXR2   | V4L2_PIX_FMT_XAVIER_SRGGB12|
+----------+-------+--------+----------------------------+
|GRGR, BGBG| 10    | JXA0   | V4L2_PIX_FMT_XAVIER_SGRBG10|
|          +-------+--------+----------------------------+
|          | 12    | JXA2   | V4L2_PIX_FMT_XAVIER_SGRBG12|
+----------+-------+--------+----------------------------+
|GBGB, RGRG| 10    | JXG0   | V4L2_PIX_FMT_XAVIER_SGBRG10|
|          +-------+--------+----------------------------+
|          | 12    | JXG2   | V4L2_PIX_FMT_XAVIER_SGBRG12|
+----------+-------+--------+----------------------------+
|BGBG, GRGR| 10    | JXB0   | V4L2_PIX_FMT_XAVIER_SBGGR10|
|          +-------+--------+----------------------------+
|          | 12    | JXB2   | V4L2_PIX_FMT_XAVIER_SBGGR12|
+----------+-------+--------+----------------------------+


TX2
------

Bit order of 10-bit and 12-bit custom formats on TX2:

+-------+---------------------------------------------------+
| Depth | Bit order (X is undefined)                        |  
+=======+===================================================+
| 10    |0 0 B9 B8 B7 B6 B5 B4 B3 B2 B1 B0 X X X X          | 
+-------+---------------------------------------------------+
| 12    |0 0 B9 B8 B7 B6 B5 B4 B3 B2 B1 B0 X X X X          |
+-------+---------------------------------------------------+

**Monochrome**

Monochrome standard format:

+-------+--------------+----------------------+
| Depth | FourCC       | Enumerator           |
+=======+==============+======================+
| 8     | GREY         | V4L2_PIX_FMT_GREY    |
+-------+--------------+----------------------+

Monochrome custom formats:

+-------+--------------+----------------------+
| Depth | FourCC       | Enumerator           |
+=======+==============+======================+
| 10    | J2Y0         |V4L2_PIX_FMT_TX2_Y10  |
+-------+--------------+----------------------+
| 12    | J2Y2         |V4L2_PIX_FMT_TX2_Y12  |
+-------+--------------+----------------------+


**RGB**

RGB3 is unsupported. X bytes are set to 0 by NVIDIA's video input unit.

+----------+-------+--------+------------------------+---------------+
|Layout    | Depth | FourCC | Enumerator             | Memory layout |
|          |       |        |                        +---------------+
|          |       |        |                        | Byte 0-1-2-3  |
+==========+=======+========+========================+===============+
| XRGB     | 8     | BX24   | V4L2_PIX_FMT_XRGB32    | B-G-R-X       |
+----------+-------+--------+------------------------+---------------+
| BGRX     | 8     | XR24   | V4L2_PIX_FMT_XBGR32    | X-R-G-B       |
+----------+-------+--------+------------------------+---------------+


**Bayer**

8-bit standard formats:

+----------+-------+--------+----------------------------+
|Pattern   | Depth | FourCC | Enumerator                 |
+==========+=======+========+============================+
|RGRG, GBGB| 8     | RGGB   | V4L2_PIX_FMT_SRGGB8        |
+----------+-------+--------+----------------------------+
|GRGR, BGBG| 8     | GRBG   | V4L2_PIX_FMT_SGRBG8        |
+----------+-------+--------+----------------------------+
|GBGB, RGRG| 8     | GBRG   | V4L2_PIX_FMT_SGBRG8        |
+----------+-------+--------+----------------------------+
|BGBG, GRGR| 8     | BGGR   | V4L2_PIX_FMT_SBGGR8        |
+----------+-------+--------+----------------------------+


10-bit and 12-bit custom formats:

+----------+-------+--------+----------------------------+
|Pattern   | Depth | FourCC | Enumerator                 |
+==========+=======+========+============================+
|RGRG, GBGB| 10    | J2R0   | V4L2_PIX_FMT_TX2_SRGGB10   |
|          +-------+--------+----------------------------+
|          | 12    | J2R2   | V4L2_PIX_FMT_TX2_SRGGB12   |
+----------+-------+--------+----------------------------+
|GRGR, BGBG| 10    | J2A0   | V4L2_PIX_FMT_TX2_SGRBG10   |
|          +-------+--------+----------------------------+
|          | 12    | J2A2   | V4L2_PIX_FMT_TX2_SGRBG12   |
+----------+-------+--------+----------------------------+
|GBGB, RGRG| 10    | J2G0   | V4L2_PIX_FMT_TX2_SGBRG10   |
|          +-------+--------+----------------------------+
|          | 12    | J2G2   | V4L2_PIX_FMT_TX2_SGBRG12   |
+----------+-------+--------+----------------------------+
|BGBG, GRGR| 10    | J2B0   | V4L2_PIX_FMT_TX2_SBGGR10   |
|          +-------+--------+----------------------------+
|          | 12    | J2B2   | V4L2_PIX_FMT_TX2_SBGGR12   |
+----------+-------+--------+----------------------------+

