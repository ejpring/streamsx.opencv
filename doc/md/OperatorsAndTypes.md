Operators and Types
===================

All of the operators and types in this toolkit are defined in the Streams namespace ’com.ibm.streamsx.opencv’. To use them in an SPL application, include this statement at the top of your SPL or SPLMM source file:

        use com.ibm.streamsx.opencv::*;

For sample applications that use these operators and types, see [Sample Applications](SampleApplications.html).

For suggestions on using the tookit, see [Using the OpenCV Toolkit](UsingToolkit.html).

For instructions on installing the toolkit and its dependencies, see [Installing the OpenCV Toolkit](InstallingToolkit.md).

The Streams types and operators in this toolkit are ’wrappers’ for C/C++ data structures, functions, and classes defined in the FFmpeg and OpenCV libraries. For further information on the OpenCV API, please refer to the PDF files installed in /usr/local/share/OpenCV/doc and the [online OpenCV documentation](http://opencv.org/documentation.html). For further information on the FFmpeg API, please refer to the ’ffmpeg’ man pages and the [online FFmpeg documentation](http://ffmpeg.org/documentation.html).

IplImage and BoxList types
--------------------------

All of the operators in this toolkit use the Streams type IplImage to represent video frames in tuples. It is defined like this:

        type IplImage = 
            int32 ipl_channels, 
            int32 ipl_depth, 
            int32 ipl_width, 
            int32 ipl_height, 
            list<uint8> ipl_data, 
            uint64 tag ;

The Streams type IplImage is a ’wrapper’ for the . Individual images representing video frames are held in the ’ipl_data’ attribute of IplImage. The format of image data is described in the .

Some of the operators in this toolkit use the Streams type BoxList to represent rectangular areas within a video frame that are of particular interest. It is defined like this:

        type BoxList = 
            list<int32> x_list, 
            list<int32> y_list, 
            list<int32> width_list, 
            list<int32> height_list ;

An attribute of Streams type BoxList always appears in tuples along with an attribute of Streams type IplImage. The BoxList contains four integer lists that always have the same number of elements, representing the position and size of a rectangular subset of the video frame in the ’ipl_data’ attribute of IplImage.

The `Crop` and `CropBoxes` operators can produce tuples of type IplImage or CroppedIplImage:

        type CroppedIplImage = 
            int32 ipl_channels, 
            int32 ipl_depth, 
            int32 ipl_width, 
            int32 ipl_height, 
            int32 ipl_x, 
            int32 ipl_y, 
            list<uint8> ipl_data, 
            uint64 tag ;

The attributes ’ipl_x’ and ’ipl_y’ of type CroppedIplImage are assigned the original position of the cropped image. That is, ’ipl_x’ and ’ipl_y’ are the ’x’ and ’y’ values the output image had in the input image.

AddText operator
----------------

Draws the value of the parameter ’text’ on the image, in the specified color, at the specified coordinates. Newline characters in the text are handled properly, but other non-printing characters are rendered as question marks.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **text**: (required rstring) Text to add to the image. The value may be an expression, including input attributes.

- **backcolor**: (optional list of three integers) Text background box RGB value, default is transparent

- **forecolor**: (optional list of three integers) Text foreground RGB value, singleton or triplet, default is black [0,0,0]

- **size**: (optional float) Text size, default 1.0

- **thickness**: (optional float) Text thickness, default 1

- **x**: (optional integer) Text starting x position from top-left, default 20, negative wraps around

- **y**: (optional integer) Text starting y position from top-left, default 20, negative wraps around

- **widthAttribute**: (optional attribute) An input attribute containing the width of the image, in pixels. If not specified, the input type must contain an ’ipl_width’ attribute.

- **heightAttribute**: (optional attribute) An input attribute containing the height of the image, in pixels. If not specified, the input type must contain an ’ipl_height’ attribute.

- **depthAttribute**: (optional attribute) An input attribute containing the depth of the pixels, in bits. If not specified, the input attribute ’ipl_depth’ is used, if present, otherwise the value ’8’ is assumed.

- **channelsAttribute**: (optional attribute) An input attribute containing the number of color channels. If not specified, the input attribute ’ipl_channels’ is used, if present, otherwise the value ’3’ is assumed.

- **dataAttribute**: (optional attribute) An input attribute containing the pixel data. If not specified, the input type must contain an ’ipl_data’ attribute. The attribute must be of type ’list<uint8>’. The size of the list must be width*height*channels.

Used in these sample applications:

-   `addtext.spl`
-   `imgcrawler.spl`

Apply2 operator
---------------

Apply a specific OpenCV function on a set of images. The operator waits until it has received an image on each of its input ports then it applies the function ’f’ as follow:

f(...f(f(img[0],img[1]),img[2])...,img[n-1])

If too many images are recieved for a given port (for example if there is a large rate mismatch) the operator will terminate.

Input:

- number of ports: >=2
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are NOT preserved from the input tuple and will be unset
- punctuation: Generating (one punctuation for each occurrence of one or more punctuations received on all input ports, i.e. non-preserving and no counts per port are kept)

Parameters:

- **transform**: (required rstring) The CV function to apply, suitable functions are 'cvOr', 'cvAnd', 'cvAbsDiff', etc.

Used in these sample applications:

-   `codebook.spl`
-   `demo2b.spl`
-   `demo2.spl`
-   `demo3b2.spl`
-   `demo3b.spl`
-   `demo3.spl`
-   `demo4b.spl`
-   `demo4.spl`
-   `drawcontoursoncam.spl`
-   `drawcontoursonurl.spl`
-   `fun.spl`
-   `mixcamsnow.spl`
-   `onc2.spl`
-   `threshold.spl`

Average operator
----------------

Compute the average luminosity of all the pixels in the image and print the result on the image. If the input image is RGB, it will first be converted to black&white before computing the average. The output tuple contains a copy of the original image (RGB or B&W) with the average value.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters: none

Used in these sample applications:

-   `equalizehist.spl`

BitBucket operator
------------------

Track number of tuples and their combined size from a particular stream. Upon receiving a punctuation on the stream, the stats {tuples/s, bytes/s, and number of seconds} are written to a log. A reset of time, #tuples, and #bytes occurs with each punctuation also.

Input:

- number of ports: 1
- tuple schema: any

Output: none

Parameters: none

Used in these sample applications:

-   `contoursonjpg.spl`
-   `speed.spl`

Blank operator
--------------

Generate blank frames of a given size and depth, with a specific color. The rate at which the frames are sent is specified. It is possible to limit how many frames are sent total.

Input: none

Output:

- number of ports: 1
- tuple schema: IplImage
- punctuation: Generating (WindowMarker occurs when requested frame count was met)

Parameters:

- **channels**: (optional integer) number of channels, 1 for black&white frames and 3 for RGB frames, default 1

- **color**: (optional integer) solid singleton or triplet color input instead of image input, default black(0)

- **count**: (optional integer) Number of output frames desired (count), default undefined (unlimited)

- **fps**: (optional integer) fps (0 means no delay), default 20

- **height**: (optional integer) height, default 480

- **image**: (optional rstring) image input instead of solid color, default undefined (use solid color)

- **width**: (optional integer) width, default 640

Used in these sample applications:

-   `blank2.spl`
-   `blank.spl`
-   `contoursonjpg.spl`
-   `imgcrawler.spl`
-   `speed.spl`

Canny operator
--------------

Apply the ’cvCanny" function to the input image and send the result out. This operation can be used to detect contours of objects in the image.

Input:

- number of ports: 1 (if TrackBars are used then could be up to 3)
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **threshold1**: (optional integer) Specify threshold1 value for cvCanny if trackbar is not used, default use TrackBar value

- **threshold2**: (optional integer) Specify threshold2 value relative to threshold1 value for cvCanny if trackbar is not used, default use TrackBar value. e.g. if threshold values are (10, 20) then parameters should be (10, 10).

- **aperture**: (optional integer) aperture size for cvCanny, default 3

Used in these sample applications:

-   `onc5.spl`

CaptureFromCAM operator
-----------------------

Capture video frames using a camera attached to the host. OpenCV supports V4Linux and FireWire on Linux. The list of supported cameras depends on the version of Linux and the type of device. Cameras that are compatible with the UVC standard are good candidates since they do not require a specific driver to be supported.

Input: none

Output:

- number of ports: 1
- tuple schema: IplImage
- punctuation: none

Parameters:

- **index**: (optional integer) cam index to read from, default 0

- **fps**: (optional integer) desired fps, default undefined (use camera queried rate or 20 fps)

- **width**: (optional integer) desired width, default undefined (use camera default)

- **height**: (optional integer) desired height, default undefined (use camera default)

Used in these sample applications:

-   `camproxy.spl`
-   `cam.spl`
-   `demo1.spl`
-   `demo2b.spl`
-   `demo2.spl`
-   `demo3.spl`
-   `demo6.spl`
-   `demo7.spl`
-   `drawcontoursoncam.spl`
-   `equalizehist.spl`
-   `facedetect.spl`
-   `fun.spl`
-   `log.spl`
-   `mixcamsnow.spl`
-   `modec.spl`
-   `onc3.spl`
-   `onc4.spl`
-   `onc5.spl`
-   `proxy2.spl`
-   `rank.spl`
-   `rotate.spl`

CaptureFromFile operator
------------------------

Read video frames from a file. The list of supported formats varies depending on the options used during the build of OpenCV and FFmpeg.

Input: none

Output:

- number of ports: 1
- tuple schema: IplImage
- punctuation: Generating (WindowMarker occurs when end of file is reached)

Parameters:

- **file**: (required rstring) file input (recommend using an absolute path)

- **rate**: (optional integer) desired frame rate, default defined in file

- **repeat**: (optional integer) repeat indefinitely, default false(0)

Used in these sample applications:

-   `addtext.spl`
-   `codebook.spl`
-   `contoursonfile.spl`
-   `copy.spl`
-   `demo1b.spl`
-   `demo2b.spl`
-   `demo3b2.spl`
-   `demo3b.spl`
-   `demo4b.spl`
-   `demo5b.spl`
-   `demo6b.spl`
-   `demo7b.spl`
-   `fileproxy.spl`
-   `file.spl`
-   `onc1.spl`
-   `onc2.spl`
-   `oncf.spl`
-   `record.spl`
-   `threshold.spl`
-   `videostream.spl`
-   `xcombine.spl`

CaptureFromURL operator
-----------------------

Obtain successive video frames from a HTTP server. The URL provided will be queried periodically to obtain the next frame. This is ideal to get video frames from an IP camera or a traffic camera. libCURL is used to retrieve the images. Please refer to libCURL documentation for the supported URL formats.

The format of the image fetched can be any format supported by OpenCV. This operator is not able to process web pages. The URL must be an image. The same URL is used to query successive frames and it is assumed that the image changes periodically on the server.

Input: none

Output:

- number of ports: 1
- tuple schema: IplImage
- punctuation: none

Parameters:

- **url**: (required rstring) Location of image to poll periodically

- **dieonerror**: (optional integer) Stop polling on error(1) or continue on errors(0), default 1

- **format**: (optional rstring) Image format, default JPG

- **rate**: (optional integer) Rate to poll the url, default 15

Used in these sample applications:

-   `box.spl`
-   `drawcontoursonurl.spl`
-   `url.spl`

CodeBook operator
-----------------

Compute the foreground/background segmentation of each frames using the CodeBook algorithm described in the book **Learning OpenCV”. This operator learns about the background for a period of time and builds a statistical model of each pixel. Once the learning phase ends, it uses the model to label each pixel of subsequence frames. A pixel belongs either to the forground or background. The result is a binary image where each pixel is either ’0’ (background) or ’255’ (foreground). Once the learning phase ends, the statistical models are not updated.

This algorithm is more powerful than a typical frame difference since the model built can accomodate periodic variation like water falls, tree leaves movement, etc. However during the learning phase, there must not be any foreground object in the field.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **banner**: (optional integer) Produce an empty frame with the text **learning in progress” for each incoming frame during the learning phase (true-1, false-0), default 0

- **time**: (optional integer) Learning time for image background in seconds (longer is better, so long as there are no foreground objects), default 10

Used in these sample applications:

-   `codebook.spl`
-   `demo4b.spl`
-   `demo4.spl`
-   `onc3.spl`

Collage operator
----------------

Assemble several video streams in a side by side fashion and produce a new video stream. Each video stream must be received on a different port. The height, depth and number of channels of the frame of each video stream must be identical. The frames are assembled from left to right. Each incoming video stream must be producing frames at the same rate. The operator will wait for a frame on each port before emitting a new composite frame.

               +----------+----------+----------+
               |          |          |          |
               |  img #1  |  img #2  |  img #3  |
               |          |          |          |
               +----------+----------+----------+

Input:

- number of ports: >=2
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are NOT preserved from the input tuple and will be unset
- punctuation: Generating (one punctuation for each occurrence of one or more punctuations received on all input ports, i.e. non-preserving and no counts per port are kept)

Parameters: none

Used in these sample applications:

-   `contoursonfile.spl`
-   `demo3b.spl`
-   `equalizehist.spl`
-   `log.spl`
-   `onc5.spl`
-   `rotate.spl`

CombineImg operator
-------------------

Overlay an image over another. The result image size is always the same as the size of the image on the first input port. The size of the second image may be bigger or smaller than the first image. The result is computed by computing the merge value of all the pixels overlapping between both images. The pixels from the second image that are outside the overlaping zone are ignored. The pixels of the first image that are not in the overlaping zone are unchanged. The opacity can be adjusted between 0 and 100 of the second image.

                 +----------------------------------------+
                 |                                        |
                 |                  img #1                |
                 |                                        |
                 |                                        |
                 |           + - - - - - - - - - - - - - -+- - - - +
                 |                                        |
                 |           |                            |        |
                 |                                        |
                 |<-offset ->|      [overlap zone]        |        |
                 |                                        |
                 |           |                            |        |
                 |                                        |
                 +-----------+----------------------------+        |

                             |                img #2               |

                             + - - - - - - - - - - - - - - - - - - +

Input:

- number of ports: 2
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are NOT preserved from the input tuple and will be unset
- punctuation: Generating (one punctuation for each occurrence of one or more punctuations received on all input ports, i.e. non-preserving and no counts per port are kept)

Parameters:

- **offset**: (optional integers) image offset (x,y) from top-left corner of image 1 and top-left corner of image 2 (can be positive or negative), default 0,0

- **opacity**: (optional integer) opacity of the second image 0(invisible) vs. 100 (completely covers), default 100

Used in these sample applications:

-   `imgcrawler.spl`
-   `xcombine.spl`

ConnectedComponents operator
----------------------------

Compute the contours of each frames then approximate contours into individual components discarding small components. This uses an algorithm described in the book **Learning OpenCV”.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **hull**: (optional integer) Smooth edges using convex hull (true-1 or false-0), Default 0 thus polygon approximation used

- **perim**: (optional float) Perimeter threshold factor, Default 4.0

Used in these sample applications:

-   `codebook.spl`
-   `demo4b.spl`
-   `demo4.spl`
-   `onc2.spl`
-   `onc3.spl`
-   `onc4.spl`

Crop operator
-------------

Crops the input image as specified by the ’topleft’ and ’bottomright’ parameters, and outputs the cropped image. Cropping rectangle must be inside of the input image.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: CroppedIplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **topleft**: (required integers) The top-left point {int,int} of crop box. The value may be an expression, including input attributes.

- **bottomright**: (required integers) the bottom-right point {int,int} of crop box. The value may be an expression, including input attributes.

Used in these sample applications:

-   `rotate.spl`

CropBoxes operator
------------------

Crops the input image as specified by the input BoxList (a list of zero or more rectangles), and outputs zero or more cropped images. Cropping rectangles must be inside of the input image.

Input:

- number of ports: 1
- tuple schema: IplImage, BoxList

Output:

- number of ports: 1
- tuple schema: CroppedIplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters: none

Used in these sample applications:

-   `facedetect.spl`

CvtColor operator
-----------------

Convert the color of the input image. Constants for color conversion are in cv.h header file. Common constants include: 'CV_GRAY2BGR', 'CV_BGR2GRAY', and 'CV_RGB2GRAY'.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **cvt_param**: (required rstring) Constants for color conversion (cv.h) (quoted)

- **channels**: (optional integer) Number of channels in the output image (1 or 3), Default 1

Used in these sample applications:

-   `codebook.spl`
-   `contoursonfile.spl`
-   `contoursonjpg.spl`
-   `demo2b.spl`
-   `demo2.spl`
-   `demo3b2.spl`
-   `demo3b.spl`
-   `demo3.spl`
-   `demo4b.spl`
-   `demo4.spl`
-   `drawcontoursoncam.spl`
-   `drawcontoursonurl.spl`
-   `fun.spl`
-   `mixcamsnow.spl`
-   `onc2.spl`
-   `onc4.spl`
-   `onc5.spl`
-   `threshold.spl`

Dilate operator
---------------

Dilates the source image using cvDilate function.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **iteration**: (optional integer) number of iterations, default 1

Used in these sample applications:

-   `demo2b.spl`
-   `demo2.spl`
-   `fun.spl`

DisplayProxy operator
---------------------

Output the image stream over a socket connection on a specific port. There are two operation modes, one that sends to a specific hostname or one that listens for an incoming connection and then sends on that connection. Hence either a hostname parameter must be set or the listen parameter set to true.

Input:

- number of ports: 1
- tuple schema: IplImage

Output: none

Parameters:

- **port**: (required integer) port

- **hostname**: (optional rstring) hostname, default undefined (listening for incoming connections must be set to true)

- **listen**: (optional boolean) listen for connections, default false (hostname must be set)

- **compress**: (optional boolean) compress before output, default false

Used in these sample applications:

-   `camproxy.spl`
-   `fileproxy.spl`
-   `proxy2.spl`

DoLog operator
--------------

Logarithmic transform of the input image (pixel-by-pixel). Common use is to enhance low pixel values at the expense of loss of information in the high pixel values.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **invert**: (optional integer) invert pixel values prior to log scaling true(1) or false(0), default 0

Used in these sample applications:

-   `log.spl`

DrawCircles operator
--------------------

Draws circles at a the set of locations specified in the BoxList. Centers of the circles are determined by taking the (x,y) coordinate and adding 1/2 the width or height respectively. Radius is 1/4 the sum of width and height.

Input:

- number of ports: 1
- tuple schema: IplImage, BoxList

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **color**: (optional integer(s)) circle color singleton or triplet, defaults 0,0,255 (first circle), 0,128,255 (second circle), 0,255,255 (third), 0,255,0 (fourth), 255,128,0 (fifth), 255,255,0 (sixth), 255,0,0 (seventh), 255,0,255 (eighth), repeat pattern for additional default colors

- **thickness**: (optional integer) thickness of circle, default 1

Used in these sample applications:

-   `demo5b.spl`
-   `demo5.spl`

DrawContours operator
---------------------

Draws the input image contours in white on top of a black output image.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters: none

Used in these sample applications:

-   `contoursonfile.spl`
-   `contoursonjpg.spl`
-   `demo3b2.spl`
-   `demo3b.spl`
-   `demo3.spl`
-   `drawcontoursoncam.spl`
-   `drawcontoursonurl.spl`
-   `fun.spl`
-   `onc4.spl`

DrawRectangles operator
-----------------------

Draws rectangles at a the set of locations specified in the BoxList. Top-left corner is determined by taking the (x,y) coordinate. The bottom-right corner is determined by adding the width or height respectively.

Input:

- number of ports: 1
- tuple schema: IplImage, BoxList

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **color**: (optional list of three integers) rectangle color, expressed as a list of integers of the form [red,green,blue] in the range of zero to 255, defaulting to [0,0,255] for the first rectange, then , [0,128,255], [0,255,255], [0,255,0], [255,128,0], [255,255,0], [255,0,0], [255,0,255], for subsequent rectanges, repeating as necessary for all rectanges

- **thickness**: (optional integer) thickness of rectangle, default 1

Used in these sample applications:

-   `facedetect.spl`

EqualizeHist operator
---------------------

Equalizes the histogram of each channel in the source image using cvEqualizeHist function.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters: none

Used in these sample applications:

-   `equalizehist.spl`

Erode operator
--------------

Erodes the source image using cvErode function.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **iteration**: (optional integer) number of iterations, default 1

Used in these sample applications:

-   `demo2b.spl`
-   `demo2.spl`

FaceDetection operator
----------------------

Uses a face model to detect faces in images, then outputs the image with a list of boxes (one for each face found).

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, BoxList, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **facemodel**: (required rstring) facemodel location (recommend using an absolute path)

- **scale**: (required float) scale input image before detection (larger values scale down the image and smaller values enlarge, 1.0 for original size), default 1.0

Used in these sample applications:

-   `demo5b.spl`
-   `demo5.spl`
-   `facedetect.spl`

Flip operator
-------------

Flips images horizontal, vertical, or both.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **horizontal**: (optional boolean) Flip image horizontal, default false

- **vertical**: (optional boolean) Flip image vertical, default false

Used in these sample applications:

-   `onc3.spl`
-   `record.spl`

FramePacer operator
-------------------

Paces the images on the input port when sending out the output port. This is either based on a rate from an input video file or based on an integer parameter.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **file**: (optional rstring) Use the frame rate from the file specified, default use rate parameter

- **rate**: (optional integer) Specify the desired output frame rate per second, default use the file parameter

Used in these sample applications:

-   `onc1.spl`
-   `onc4.spl`

HTTPViewer operator
-------------------

HTTP Server serving images at a static location. As new image tuples arrive the served image is changed.

If viewed using a browser, clicking refresh should update the displayed image.

Input:

- number of ports: 1
- tuple schema: IplImage

Output: none

Parameters:

- **httplabel**: (optional rstring) Register HTTP server with label, default undefined

- **httpport**: (optional integer) http port used for output, default use the system default

Used in these sample applications:

-   `blank2.spl`

MakeBox operator
----------------

Draws rectangles at a the set of locations specified in the BoxList. Top-left corner is determined by taking the (x,y) coordinate. The bottom-right corner is determined by adding the width or height respectively.

Input:

- number of ports: 1
- tuple schema: IplImage, BoxList

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **bottomright**: (required integers) Coordinate pair for location of box bottom-right corner x,y

- **color**: (required integer(s)) RGB singleton or triplet color of the box

- **topleft**: (required integers) Coordinate pair for location of box top-left corner x,y

- **opacity**: (optional integer) Opacity percent of the box[0,100], default 100

Used in these sample applications:

-   `box.spl`
-   `onc1.spl`

MJPEGStreamer operator
----------------------

HTTP Server streaming images at a static location. As new image tuples arrive the image is sent to connected clients.

If viewed using a browser, the image should automatically update as new images arrive.

Input:

- number of ports: 1
- tuple schema: IplImage

Output: none

Parameters:

- **httplabel**: (optional rstring) Register HTTP server with label, default undefined

- **httpport**: (optional integer) http port used for output, default use the system default

Used in these sample applications:

-   `blank.spl`
-   `demo1b.spl`
-   `demo2b.spl`
-   `demo3b2.spl`
-   `demo3b.spl`
-   `demo4b.spl`
-   `demo5b.spl`
-   `demo6b.spl`
-   `demo7b.spl`
-   `onc1.spl`
-   `onc4.spl`
-   `videostream.spl`

MotionDetection operator
------------------------

Detects object motion in the image and draws flow arrows on image. This runs the Shi and Tomasi algorithm to detect feature points in the input image. Then using the Pyrimidal Lucas Kanade Optical Flow algorithm determine the flow of points between the previous image and the input image, resulting in a new set of feature points between the two images. Using the results flow arrows are drawn on the input image, longer arrows indicates larger movement.

Input:

- number of ports: 1 (if TrackBars are used then could be up to 4)
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **featurecount**: (optional integer) Specify featurecount value if trackbar is not used, default use TrackBar value

- **pyramidlevel**: (optional integer) Specify pyramidlevel value if trackbar is not used, default use TrackBar value

- **termeps**: (optional float) termeps, default 0.3

- **termiteration**: (optional integer) Specify termiteration value if trackbar is not used, default use TrackBar value

- **threshold**: (optional float) threshold, default 1f

Used in these sample applications:

-   `demo6b.spl`
-   `demo6.spl`
-   `demo7b.spl`
-   `demo7.spl`
-   `modec.spl`

Nop operator
------------

No operation is performed on the input image before outputing it.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters: none

Used in these sample applications:

-   `equalizehist.spl`

RankFilter operator
-------------------

Filters input image pixel-by-pixel by comparing it to its neighboring pixels within a radius. The comparison/replacement method is one of the following: minimum, median, or maximum.

The median filter is normally used to reduce noise in an image while preserving image details.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **mode**: (optional rstring) mode {min, med(ian), max}, default median

- **radius**: (optional integer) filter radius, default 3

Used in these sample applications:

-   `rank.spl`

ReadImage operator
------------------

Reads the image at the path indicated in the input tuple, then outputs the image as a tuple.

Input:

- number of ports: 1
- tuple schema: <rstring filename>

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters: none

Used in these sample applications:

-   `imgcrawler.spl`

Resize operator
---------------

Resizes the input image and outputs the result.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **height**: (optional integer) Height of resized image, cannot be used with ’percent’ parameter. If used without ’width’ parameter, aspect ratio will be kept

- **percent**: (optional integer) Percent (Integer) of resizing original image, cannot be used with ’width’ or ’height’ parameters. Less than 100 will be smaller and greater than 100 will be larger

- **width**: (optional integer) Width of resized image, cannot be used with ’percent’ parameter. If used without ’height’ parameter, aspect ratio will be kept

Used in these sample applications:

-   `snow.spl`
-   `xcombine.spl`

Rotate operator
---------------

Scales the input image and rotates the image around a center point.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **angle**: (required float) rotation angle in degrees and positive values mean counter-clockwise

- **center**: (optional integers) center point {int,int} to rotate around, default uses the center of the image

- **scale**: (optional float) isotropic scale factor. Less than 1.0 will be smaller and greater than 1.0 will be larger, default 1.0 (no scaling)

Used in these sample applications:

-   `rotate.spl`

SaveImage operator
------------------

Save each input image in an image file.

The file format is deduced from the file extension. The image formats supported depend upon the image codecs installed on the system, which probably include:

-   ’.jpeg’ and ’.jpg’, implying JPEG format
-   ’.png’, impying Portable Network Graphics format
-   ’.bmp’, implying Windows bitmap format
-   ’.pbm’, ’.pgm’, and ’.ppm’, implying portable image format

For details, see the documentation for the

Optionally, a sequential integer can be inserted in front of the file extension.

Input:

- number of ports: 1
- tuple schema: IplImage

Output (optional):

- number of ports: 1
- tuple schema: rstring, the name of the file in which the input image was saved
- punctuation: Forwarded

Parameters:

- **file**: (required rstring) The name of the file, including the extension (which implies the image format) and optionally including the directory. The value may be an expression, including input attributes.

- **increment**: (optional boolean) If specified and true, inserts a sequential integer in front of the file extension.

- **widthAttribute**: (optional attribute) An input attribute containing the width of the image, in pixels. If not specified, the input type must contain an ’ipl_width’ attribute.

- **heightAttribute**: (optional attribute) An input attribute containing the height of the image, in pixels. If not specified, the input type must contain an ’ipl_height’ attribute.

- **depthAttribute**: (optional attribute) An input attribute containing the depth of the pixels, in bits. If not specified, the input attribute ’ipl_depth’ is used, if present, otherwise the value ’8’ is assumed.

- **channelsAttribute**: (optional attribute) An input attribute containing the number of color channels. If not specified, the input attribute ’ipl_channels’ is used, if present, otherwise the value ’3’ is assumed.

- **dataAttribute**: (optional attribute) An input attribute containing the pixel data. If not specified, the input type must contain an ’ipl_data’ attribute. The attribute must be of type ’list<uint8>’. The size of the list must be width*height*channels.

Used in these sample applications:

-   `facedetect.spl`

SaveToFile operator
-------------------

Save the input images to a video file. The video file frame rate can be defined either by parameter ’rate’ or by the frame rate of another video file ’frate’.

The codec for the output video file is defined by the ’fourcc’ parameter. Common examples are:

       ##   Codec ----->  CV_FOURCC
       ##
       ##   mpeg1         PIM1
       ##   motion-jpeg   MJPG
       ##   mpeg4.2       MP42
       ##   mpeg4.3       DIV3
       ##   mpeg4         DIVX
       ##   h263          I263
       ##   flv1          FLV1

Input:

- number of ports: 1
- tuple schema: IplImage

Output: none

Parameters:

- **filename**: (required rstring) filename of output file

- **fourcc**: (required rstring) Four letter CV_FOURCC corresponding to the desired output codec

- **rate**: (optional integer) output frame rate

- **frate**: (optional rstring) set output frame rate equal to frame rate in input file(frate)

Used in these sample applications:

-   `contoursonfile.spl`
-   `copy.spl`
-   `demo3b.spl`
-   `onc5.spl`
-   `oncf.spl`
-   `record.spl`

Sequencer operator
------------------

Uses an integer tag on incoming images to order the images in ascending order.

Input:

- number of ports: 1
- tuple schema: <IplImage, tuple<uint64 tag>>

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters: none

Used in these sample applications:

-   `equalizehist.spl`

SkipEmpty operator
------------------

Discards empty frames in a stream. A frame is determined to be non-empty if the non-black pixel content is above a threshold.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **threshold**: (optional float) Threshold to determine if a frame is empty, default 0.05 (i.e. If the average pixel value across the frame is less than 0.05, then it is empty/black)

Used in these sample applications:

-   `onc1.spl`
-   `onc3.spl`
-   `onc4.spl`
-   `onc5.spl`
-   `oncf.spl`

SkipFrame operator
------------------

Discards frames in a stream. A frame is skipped either adaptively and/or fixed. If ’fixed’, a ’frame’ number of images are skipped between each processed frame. If ’adaptive’, a fixed number of images can still be skipped if the ’frame’ parameter is specified, regardless adaptive will also skip any frames when the previous frame has not finished being output yet.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **style**: (required rstring) style ('adaptive' or 'fixed')

- **frame**: (optional integer) Number of frames to skip between processing a frame, default 0

Used in these sample applications:

-   `rank.spl`

Smooth operator
---------------

Use guaussian smoothing to smooth images.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **iteration**: (required integer) Number of times to apply guaussian smoothing to images in-place, default 1

Used in these sample applications:

-   `codebook.spl`
-   `contoursonfile.spl`
-   `contoursonjpg.spl`
-   `demo2b.spl`
-   `demo2.spl`
-   `demo3b2.spl`
-   `demo3b.spl`
-   `demo3.spl`
-   `demo4b.spl`
-   `demo4.spl`
-   `drawcontoursoncam.spl`
-   `drawcontoursonurl.spl`
-   `fun.spl`
-   `onc3.spl`
-   `snow.spl`
-   `speed.spl`

Snow operator
-------------

Generate random **snow” frames consisting of either black or white pixels. The rate at which the frames are sent is specified.

Input: none

Output:

- number of ports: 1
- tuple schema: IplImage
- punctuation: none

Parameters:

- **fps**: (optional integer) Number of frames per second to be generated, default 20

- **height**: (optional integer) Height of snow image to be generated, default 480

- **width**: (optional integer) Width of snow image to be generated, default 640

Used in these sample applications:

-   `mixcamsnow.spl`
-   `snow.spl`

Sobel operator
--------------

Apply the Sobel edge detection function to the input image, and output the resulting image.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage
- punctuation: none

Parameters: none

Used in these sample applications:

-   `facesedgescam.spl`
-   `facesedgesfile.spl`

SourceProxy operator
--------------------

Input the image stream over a socket connection on a specific port. There are two operation modes, one that receives from a specific hostname or one that listens for an incoming connection and then receives on that connection. Hence either a hostname parameter must be set or the listen parameter set to true.

Input: none

Output:

- number of ports: 1
- tuple schema: IplImage
- punctuation: none

Parameters:

- **port**: (required integer) port

- **banner**: (optional boolean) Use banner image while waiting for incoming connection, default false

- **hostname**: (optional rstring) hostname, default undefined (listening for incoming connections must be set to true)

- **listen**: (optional boolean) listen for connections, default false (hostname must be set)

Used in these sample applications:

-   `demo4.spl`
-   `demo5.spl`
-   `fileproxy.spl`
-   `proxy2.spl`
-   `srcproxy.spl`

Stats operator
--------------

Prints incoming stream stats on images before outputing. Parameters allow for specifying which stats are printed, by default all available stats are printed (FPS, KBs, image size, number of channels, and count.)

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **channels**: (optional boolean) track channels stat, default true

- **count**: (optional boolean) track count stat, default true

- **fps**: (optional boolean) track fps stat, default true

- **kbs**: (optional boolean) track kbs stat, default true

- **size**: (optional boolean) track size stat, default true

Used in these sample applications:

-   `blank2.spl`
-   `blank.spl`
-   `demo6b.spl`
-   `demo6.spl`
-   `demo7b.spl`
-   `demo7.spl`
-   `fun.spl`
-   `modec.spl`
-   `rank.spl`

Tagger operator
---------------

Tags incoming images with an ascending integer tag.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: <IplImage, tuple<uint64 tag>>, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters: none

Used in these sample applications:

-   `equalizehist.spl`

Threshold operator
------------------

Applies a fixed-level threshold to input image. Images must be single-channel, 8-bit images. Threshold types can be found in cv.h, common types are: 'CV_THRESH_TOZERO', 'CV_THRESH_BINARY' and 'CV_THRESH_TOZERO_INV'

Input:

- number of ports: 1 (if a TrackBar is used then could be up to 2)
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **max**: (required integer) the maximum value to use with thresholding types

- **threshold_type**: (required rstring) Types of thresholding (cv.h) (quoted)

- **threshold**: (optional integer) Specify threshold value if trackbar is not used, default use TrackBar value

Used in these sample applications:

-   `contoursonfile.spl`
-   `contoursonjpg.spl`
-   `demo2b.spl`
-   `demo2.spl`
-   `demo3b2.spl`
-   `demo3b.spl`
-   `demo3.spl`
-   `drawcontoursoncam.spl`
-   `drawcontoursonurl.spl`
-   `fun.spl`
-   `onc2.spl`
-   `onc4.spl`
-   `threshold.spl`

TimeStamp operator
------------------

Prints a timestamp on input images before outputting.

Input:

- number of ports: 1
- tuple schema: IplImage

Output:

- number of ports: 1
- tuple schema: IplImage, any additional attributes in the output tuple are assumed to be preserved from the input tuple
- punctuation: Forwarded

Parameters:

- **relative**: (optional integer) Relative from application start(non-zero) or system time(0), default 0 for system time

Used in these sample applications:

-   `file.spl`
-   `record.spl`

TrackBar operator
-----------------

Creates a Trackbar to allow users to input a sliding integer value at runtime. The Trackbar can attach to an existing window or be in a window of its own.

This operator is currently setup to interact with 'Canny', 'MotionDetection', and 'Threshold' operators. Each of these operators require integer inputs either via a Trackbar or a parameter. To use a Trackbar these operators expect the tuple to contain an integer parameter whose name is 'trackbar_{parameter name}' (replacing 'parameter name' with the appropriate name, e.g. 'trackbar_threshold1' or 'trackbar_featurecount')

Input: none

Output:

- number of ports: 1
- tuple schema: <int32 trackbar_{name}>
- punctuation: none

Parameters:

- **display**: (required rstring) Graphical Display, typically $DISPLAY

- **max**: (required integer) Maximum trackbar value

- **name**: (required rstring) Trackbar name

- **value**: (required integer) Initial trackbar value

- **title**: (optional rstring) title of display window, if attaching to an existing window, the titles must match.

Used in these sample applications:

-   `demo2b.spl`
-   `demo2.spl`
-   `demo3b2.spl`
-   `demo3b.spl`
-   `demo3.spl`
-   `demo7b.spl`
-   `demo7.spl`
-   `fun.spl`
-   `onc2.spl`
-   `onc4.spl`
-   `onc5.spl`
-   `threshold.spl`

X11Viewer operator
------------------

Displays incoming images in an X11 window.

Input:

- number of ports: 1
- tuple schema: IplImage

Output: none

Parameters:

- **display**: (optional rstring) which graphical display to use, default is the environment variable $DISPLAY

- **skip**: (optional integer) skip n frames between each displayed frame, default 0 (skip none)

- **title**: (optional rstring) title of display window, default is the operator name

- **widthAttribute**: (optional attribute) An input attribute containing the width of the image, in pixels. If not specified, the input type must contain an ’ipl_width’ attribute.

- **heightAttribute**: (optional attribute) An input attribute containing the height of the image, in pixels. If not specified, the input type must contain an ’ipl_height’ attribute.

- **depthAttribute**: (optional attribute) An input attribute containing the depth of the pixels, in bits. If not specified, the input attribute ’ipl_depth’ is used, if present, otherwise the value ’8’ is assumed.

- **channelsAttribute**: (optional attribute) An input attribute containing the number of color channels. If not specified, the input attribute ’ipl_channels’ is used, if present, otherwise the value ’3’ is assumed.

- **dataAttribute**: (optional attribute) An input attribute containing the pixel data. If not specified, the input type must contain an ’ipl_data’ attribute. The attribute must be of type ’list<uint8>’. The size of the list must be width*height*channels.

Used in these sample applications:

-   `addtext.spl`
-   `box.spl`
-   `cam.spl`
-   `codebook.spl`
-   `contoursonfile.spl`
-   `contoursonjpg.spl`
-   `copy.spl`
-   `demo1.spl`
-   `demo2.spl`
-   `demo3.spl`
-   `demo4.spl`
-   `demo5.spl`
-   `demo6.spl`
-   `demo7.spl`
-   `drawcontoursoncam.spl`
-   `drawcontoursonurl.spl`
-   `equalizehist.spl`
-   `facedetect.spl`
-   `fileproxy.spl`
-   `file.spl`
-   `fun.spl`
-   `imgcrawler.spl`
-   `log.spl`
-   `mixcamsnow.spl`
-   `modec.spl`
-   `onc2.spl`
-   `onc3.spl`
-   `onc5.spl`
-   `proxy2.spl`
-   `rank.spl`
-   `record.spl`
-   `rotate.spl`
-   `snow.spl`
-   `srcproxy.spl`
-   `threshold.spl`
-   `url.spl`
-   `xcombine.spl`

----

&copy; Copyright 2012, 2016, International Business Machines Corporation, All Rights Reserved 
