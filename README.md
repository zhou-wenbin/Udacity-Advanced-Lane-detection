## **Advanced Lane Detection** 

### In this project, I will demonstrade an advanced lane detection algorithm compared to previous one. 

---


The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./camera_cal/calibration2.jpg "distorted"
[image2]: ./output_images/undistorted_of2.png "undistorted"
[image3]: ./test_images/test5.jpg "test5"
[image4]: ./output_images/undistorted_of_test5.png "undistorted of test5"
[image5]: ./test_images/straight_lines2.jpg 
[image6]: ./output_images/processed_of_test5.jpg
[image7]: ./output_images/processed_of_straight_lines2.jpg
[image8]: ./output_images/S_channel_of_test5.jpg
[image9]: ./output_images/H_channel_of_test5.jpg
[image10]: ./output_images/L_channel_of_test5.jpg
[image11]: ./output_images/processed_of_test5.png
[image12]: ./output_images/masked_of_test5.png
[image13]: ./output_images/warped_of_test5.png
[image14]: ./output_images/warped_binary_of_test5.png
[image15]: ./output_images/pixels_of_warped_test5.png
[image16]: ./output_images/fit_polynomial_in_warped_test5.png
[image17]: ./output_images/histogram_of_warped_test5.png
[image18]: ./output_images/test_result.png











### Here I will consider the [rubric](https://review.udacity.com/#!/rubrics/571/view) points individually and describe how I addressed each point in my implementation.  

---

### 1.Camera calibration

#### 1.1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the IPython notebook located in "./camera calibrating.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then use the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image in "./camera_cal/calibration2.jpg" using the `cv2.undistort()` function. For later usage, I will save camera calibration and distortion coefficients in a file located in "./camera_cal/wide_dist_pickle.p".
* The distorted and undistorted testing images are shown as follows:

Distorted             |  Undistorted
:-------------------------:|:-------------------------:
![alt text][image1]  |  ![alt text][image2]

* Let us try to distort a road image taken by the camera:

Distorted             |  Undistorted
:-------------------------:|:-------------------------:
![alt text][image3]  |  ![alt text][image4]

---

### 2. Image processing with color transform and gradient thresholds

The code for this step is contained in the IPython notebook named "Explanation of the image process steps by steps.ipynb"

#### 2.1 Color channel choosing

In the task of lane detection when the shodow of road trees are projected onto the road, it becomes a bit more difficult to detect the lane lines that are covered in the shadow. However, when we do the color transfor of the image, we will overcome this issue. In what follows, I will explain in details.

##### 2.1.1 The bottleneck to detect lane lines by only using gradiant of an image

Normally, it is possible to detect lane lines from an image by shresholding the gradient in the x direction since we have the common sense that the lane lines are almost perpendicular to the y direction of an image. In the following two unprocessed images if we use same shreshold of x gradient to detect lane lines we will see that the image1 with shadow performs worse. In particular, those lines covered in the shadow cannot be detected.


Image1             |  Image2
:-------------------------:|:-------------------------:
![alt text][image3]  |  ![alt text][image5]


Processed Image1             |  Processed Image2
:-------------------------:|:-------------------------:
![alt text][image6]  |  ![alt text][image7]

##### 2.1.2 Color transform
In what follows, I will transform images to HLS color space (hue, lightness, and saturation), which is one of the most commonly used color spaces in image analysis. To get some intuition about these color spaces, you can generally think of Hue as the value that represents color independent of any change in brightness. So if you imagine a basic red paint color, then add some white to it or some black to make that color lighter or darker -- the underlying color remains the same and the hue for all of these colors will be the same.

On the other hand, Lightness and Value represent different ways to measure the relative lightness or darkness of a color. For example, a dark red will have a similar hue but much lower value for lightness than a light red. Saturation also plays a part in this; saturation is a measurement of colorfulness. So, as colors get lighter and closer to white, they have a lower saturation value, whereas colors that are the most intense, like a bright primary color (imagine a bright red, blue, or yellow), have a high saturation value. You can get a better idea of these values by looking at the 3D color spaces pictured below.

Next, let's us apply this theory to a testing image. First, let's take a look at the three channels of an HIS space image. 


S channle of Image             |  H channel of  Image  |  L channel of  Image
:-------------------------:|:-------------------------:|:-------------------------:
![alt text][image8]  |  ![alt text][image9] |![alt text][image10] 


From the above, we notice that S channel highlights the yellow line very well, so we will use the channel to do the shreshold manipulation. 

#### 2.2 Combine color channel and x gradient to shreshold the image. 

We will combine the shreholds discussed above to process image to find lane lines (typically the yellow lines). The following two images are the contrast. We set the shresholds as follows by test and trials:

| gradient shreshold        |    S channel shreshold   | 
|:-------------:|:-------------:| 
| 20,100     | 170,255       | 

We show the images as follows after the thresholds, since it looks well in detecting the yellow line covered in the shadow, we will use the combined shresholds to process images.

Original             |  Processed
:-------------------------:|:-------------------------:
![alt text][image3]  |  ![alt text][image11]






### 3. Get birdseye view of warped lines in an image

In order to draw lines on the warped lane lines on the road, we could firstly find those warped lines through birdseye view of an image.  This can be done by selecting the specific area where the warped lines. Note that the source area and destination area were both tested in the notebook.

Original             |  Warped
:-------------------------:|:-------------------------:
![alt text][image12]  |  ![alt text][image13]

### 4.  Find lane-line pixels and fit their positions with a polynomial in the warped image

*  4.1 Let us draw the warped binary image in birdseye view then draw its histogram.

warped_binary             |  histogram  |  
:-------------------------:|:-------------------------:|
![alt text][image14]  |  ![alt text][image17] |


From above images, we see that the peak in the histogram represents the place with pixels in the warped binary image. By using histogram of an image, we can find the lane pixels in the image, shown in the following image with 9 square windows on each side of the lane. This following video explains how to find the pixels. 
[![Click](https://youtu.be/siAMDK8C_x8/0.jpg)](https://youtu.be/siAMDK8C_x8) 

After find the pixels of the lane, we can then draw polynomial lines on the pixels centers.

pixels_location             |  fit polynomial  |  
:-------------------------:|:-------------------------:|
![alt text][image15]  |  ![alt text][image16] |

### 5. Determine the curvature of the lane and vehicle position with respect to center.
Given the polynomial line, we can compute the radius of curvature of the fit. Be careful that we will use the polynomial as x=f(y), since pixels coordinates in x direction are almost fixed. Given x=f(y), we can compute the radius of curvature easily through a well defined equation: Radius of curvature= [1+(dy/dx)^2]^{3/2}/(|d^2y/dx^2|).
### 6. Warp the detected lane boundaries back onto the original image.

Use the `cv2.warpPerspective()` funciton we can get unwarped image using inverse perspective matrix, which was calculated in the `warp_image_to_birdseye_view()` function.

### 7. Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

Finally, we use `cv2.putText()` function to write calculated numbers and words on the image. The following is a testing image.

![alt text][image18]

### 8. Combine everything, we conclude with the followings pipeline.

The code of all combined together is in the IPython notebook named "pipeline.ipynb" and you can also find the code in "functions.py"

*  Calibrate the camera;
*  Get undistorted image;
*  Set parameters;
*  Get shresholded image;
*  Get birdseye view of warp image;
*  Draw_lane_lines.




```python

```
