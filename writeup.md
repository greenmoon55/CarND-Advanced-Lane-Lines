##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Advanced Lane Finding Project**

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

[image1]: ./images/undistort.png
[image2]: ./images/undistort_road.png "Road Transformed"
[image3]: ./images/threshold_pipeline.png "Binary Example"
[image4]: ./images/perspective_transform_orig.png "Warp Example"
[image5]: ./images/transformed_image_straight.png "Warp Example"
[image6]: ./images/histogram.png
[image7]: ./images/sliding_window.png
[image8]: ./images/plot_back.png
[video1]: ./output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./P4.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]
####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image.  I used absolute sobel threshold on x and y directions, then I used magnitude and direction threholds. Lastly, I selected yellow color using a HSV mask. `    combined[((gradx == 1) & (grady == 1)) |  (h_binary == 1) | ((mag_binary == 1)& (dir_binary == 1) )] = 1` Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`.  This function takes as inputs an image (`img`), as well as the transform matrix obtained by camera caliberation.  I chose the hardcode the source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 260, 680      | 320, 720      | 
| 596, 450      | 320, 0        |
| 685, 450      | 900, 0        |
| 1045, 680     | 900, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]
![alt text][image5]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

![alt text][image6]

I applied the sliding window approach. First, I identified two peaks on the histogram and set them as the center of two windows. Then I search upwards and find the center of each sliding window by averaging x values. I used a total of 9 windows. Lastly, I fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image7]

Note that the histogram is only needed in the first window and first picture, or when the last frame in the video failed to identify lane lines. After getting fitted lines, I can simply find points that are near the previously detected lines in the next video to increase processing speed.

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in `calculate_curvature` and `draw_image` methods.

Curvature is calculated by the equation provided in the course material. I also multiplied `ym_per_pix` and `xm_per_pix` to get the result in meters rather than pixels.

I calculated the lane center by averaging the left and right x values in the bottom, then compare it with the center of the picture. 

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the function `draw_image`.  Here is an example of my result on a test image:

![alt text][image8]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The most important thing in this project is to find a robust color mask or gradients get all the lane lines that we needed. However, it is very difficult to find one. I started with sobel_x and thresholding on s channel. The threshold on s worked for yellow lines, but it couldn't omit the shadow between two lanes after a lot of trial and error. In the end, I tried the current methods to explictly mask yellow, which worked well.

The second issue is that I've made many silly mistakes, such as forgetting to convert from int to float. Also it is very important to have a clear structure for debugging where lines of code should be grouped into methods or even classes. It will save a lot of debugging time.

Though I do not have time to check challenge video, I think it will likely to fail when there are shadows or steep turns. To make it more robust, I can try to average values from multiple frames and omit the curvature if it is does not meet highway specifications.
