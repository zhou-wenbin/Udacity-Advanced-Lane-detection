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


### Here I will consider the [rubric](https://review.udacity.com/#!/rubrics/571/view) points individually and describe how I addressed each point in my implementation.  

---

### 1.Camera calibration

#### 1.1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the IPython notebook located in "./camera calibrating.ipynb"。

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

The code for this step is contained in the IPython notebook located in "TBA"

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





<!-- ### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.   -->


```python

```
