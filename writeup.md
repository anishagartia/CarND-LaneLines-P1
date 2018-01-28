# **Finding Lane Lines on the Road** 

### Anisha Gartia


The goals / steps of this project are the following:

* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[test]: ./examples/test.png "Test Image"
[gray]: ./examples/gray.png "Grayscale Image"
[blur]: ./examples/blur.png "Blurred Image"
[canny]: ./examples/canny.png "Canny Edges"
[mask]: ./examples/mask.png "Mask"
[masked_canny]: ./examples/masked_canny.png "Canny Edges applied with Mask"
[broken_hough]: ./examples/broken_hough.png "Hough Lines"
[extrapolated_hough]: ./examples/extrapolated_hough.png "Extrapolated Hough Lined"
[final_broken]: ./examples/final_broken.png "Hough lines on image"
[final_extrapolated]: ./examples/final_extrapolated.png "Result"

---

## Reflection

### 1. Pipeline Description

The flow of pipeline starts with a set of test images in .jpg format, and ends in display of the same image showing clearly marked lane lines. 

![alt text][test] 

The main libraries used for the computations are cv2 and numpy. cv2 provides the major tools to process the images. It also has functions for specific computer vision algorithms that are used in this project extensively.


The pipeline consists of seven major steps. Each of the steps is bundled in userdefined function. The functions are then called to create the pipeline flow. The steps and their corrsoposnding functions are:

*  *Change image to grayscale:*
 
This function takes the image as input, applies cv2 function cvtColor, and returns the image in grayscale. 

![alt text][gray]

* *Apply Gaussian Blur to the image:*

This function takes an image and kernel size as inputs, and applies Gaussian Blur on the image with the specified kernel size. The cv2 function GaussianBlur is used. The blured image is then returned. The idea behind applying a smoothening functions on the image is to smoothen out small sudden changes in intensity between pixels. Such sudden changes can be seen as edges during edge detection, hence giving us false edges. On applying Gaussian Blur, the intensity change between adjacent pixels is smooth.

![alt text][blur]

* *Find Canny Edges:*

This functions takes an image, high, and low threshold for Canny function as input, and applies the cv2 canny edge detection function on the image. The functions the returns an image with only the edges. As the thresold is changes, more or less edges are detected. Wider threshold margin will result in more edged being detected. 

![alt text][canny]

* *Mask with region of interest*

The masked region is the region of the image where the lane can be found. Typically while driving, the lane is seen ahead, in the bottom part of field of sight, and looking ahead the two edges of the lane appear to mean at the vanishing point. Corrpospondingly in the image, the region of interest is in the bottom half of the image, in a triangular shape. The triangular shape mimics the vanishing point.  

To allow some tolerance, we use a polygon as shown below that is wide at the bottom of image, and tapers near the middle. In the mask shown below, the black region is to be ignored, and white region in meant to be visible.

![alt text][mask]

The above mask is then applied to the canny edges image. The two images are simply combined with a bitwise "AND". This results in the black region to be masked, just hiding all the canny edges in that region. We are then left with the canny edges in the region of interest. In out project, this masked canny edge containes the edges corrosponding to the lane lines only as shown below.

![alt text][masked_canny]

* *Draw Hough Lines:*

The canny edges in the region of interest are passed through an userdefined Hough lines fucntion to generate hough lines. This hough lines function uses cv2's HoughLinesP function. Now we have the Hough lines in the form of coordinates of the start and end points of these lines. These coordinates can now be used to draw lines on an image. 

![alt text][broken_hough]

The function that draws the hough lines on the image is named draw_lines. The image with lines drawn on it is then returned from the hough lines function.

* *Draw Lines*

The drawlines function takes as input the test image, and hough lines coordinates. The function draws lines on a blank image using the coordinates and cv2's function 'line'. The the image with lines drawn on it is returned.

To extrapolate and draw complete lane lines on the image in-place of broken lines, we do the following processing:

1.  Calculate the slope of all the hough lines using the equation  
 $ slope = \frac{y2-y1}{x2-x1} $ where $ y2, x2, y1, x1 $ are coordinates of begining and end of each hough line.
 
2. Separate the hough lines with positive slopes and negative slopes to different arrays. Since the lane lines have opposite slopes, this is the method to distinguish between them.
 
3. Find the mean slope of all hough lines in with positive and negative slopes independently. As there are many broken lines, we wish to find one line with a single slope that we can extrapolate from start to end of lane line. The mean of all slopes will give us the best slope.

4. Delete all hough lines with slope greater than a fixed percentage of tolerace from the mean. This will help us remove erroneous edges on the lane not belonging to lane line that may give us erroneous slopes. Recalculate the mean slope. 

5. Now we have the slope and want to calculate x coordinates of points on the lane at bottom of image and at the end of the lane. Near the botton of the image the y coordiate will be the image.shape[0]. Near the end of the lane the y coordinate will be near by middle of the image. With trial and error, the best y coordinate was image.shape[0]* 0.6. The x coordinate is calculated as follows: 
 
$ x = x1 + \frac{y-y1}{slope} $  

Here {x1, y1} are coordinates of any point on the lane line, and y is the coordinate of the target point at top or bottom of lane we are calculating.

One calculating the x coordinates we can draw hough lines of high thickness extending from end of lane to the start of lane. THis is done using cv2.line function.

![alt text][extrapolated_hough]

* *Weighted Image*

This function takes as input the original image and the image with hough lines. The function does a weighted sum of both the image pixel wise to retun the original image superimposed with the hough lines. The weights of summation can be passes as inputs to the function, or are predefined. The returned image is as shown below. 

![alt text][final_extrapolated]


### 2. Potential shortcomings with your current pipeline

One potential shortcoming would be when the lane curves. At present, the lane lines are straight lines. They are pre defined to this geometrical shape and it would fail to show the correct lane when a curve is seen in the image.

Another shortcoming would be failure in displaying the correct lane when the lane line length changes. Tee current lane lines are extrapolated to a finxed lenght. If it is shorter, lane lines would be incorrect.

A third shortcoming is the presence of vehicles in front. The object if not detected and taken to account will result in lane lines possibly being drawn over it.

### 3. Suggest possible improvements to your pipeline

A possible improvement would be to take to account the radius of curvature if the hough lines before drawing straight lines.

Another potential improvement could be to detect other objects in front before drawing lane lines. 
