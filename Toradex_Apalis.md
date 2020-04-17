# Toradex Apalis iMX8 restrictions for CSI-2 cameras

## Unsupported pixel formats
Apalis iMX8 doesn't support all pixel formats of CSI-2 cameras.
RAW formats (bayer and mono) are right-shifted by 2 bits, 
so that every pixel loses 2 bits and cannot be used. This is a restriction of the board.

## Workaround
As a workaround, the Allied Vision driver sets the camera to YUV when the pixel format 
V4L2_PIX_FMT_GREY is selected by the user.
The driver then uses the ISI of Apalis iMX8 to convert the image to Mono8.
This is done by using a multiplane YUV format internally and providing only the first plane 
(the luma component) to the user space. The color components are discarded.

Because the camera is streaming with YUV, only half of the bandwidth of RAW8 can be achieved.   
For the Bayer pixel formats, no workaround is available.

For more information on the issue, see:   
https://community.nxp.com/thread/519316   
https://community.nxp.com/docs/DOC-345149   


