##Advanced Lane Finding Project 
###(Udacity Nanodegree Project 4)

---

<!-- **Advanced Lane Finding Project** -->

The goals / steps of this project are the following:

1. Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
2. Apply a distortion correction to raw images.
3. Use color transforms, gradients, etc., to create a thresholded binary image.
4. Apply a perspective transform to rectify binary image ("birds-eye view").
5. Detect lane pixels and fit to find the lane boundary.
6. Determine the curvature of the lane and vehicle position with respect to center.
7. Warp the detected lane boundaries back onto the original image.
8. Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/undistort_result1.png "Result of Distortion Correction on Chessboard Image"
[image2]: ./output_images/undistort_result2.jpg "Result of Distortion Correction on Road Image"
[image3]: ./output_images/rgb_colourspace.jpg "Test Image Shown in RGB Colour Space"
[image4]: ./output_images/hls_colourspace.jpg "Test Image Shown in HLS Colour Space"

[image5]: ./output_images/rgb_white_threshold.png " "
[image6]: ./output_images/rgb_yellow_threshold.png " "
[image7]: ./output_images/hls_yellow_threshold.png " "
[image8]: ./output_images/warp_verify.png "Perspective Transform Output"
<!-- [video1]: ./output_images/rgb_yellow_threshold.png " " -->

#### This writeup explains the steps I followed in my implementation by following the [Rubric Points](https://review.udacity.com/#!/rubrics/571/view)

---
<!-- ###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point. -->  

<!-- You're reading it! -->
###1.0 Camera Calibration

####Computation of camera calibration matrix and distortion coefficients given a set of chessboard images

The code for this step is contained in the second code cell of the IPython notebook located in "pipeline_on_test_images.ipynb"
Here, I prepared 'object points' which are points in real world space (x, y, z) coordinates of the chessboard corners. Since we assume the chessboard is on a flat plane, then z=0 so we only consider (x, y). To do this, I went through each chessboard image in the image folder and use the opencv function cv2.findChessboardCorners() to find the corners of the chessboard. I then append these corners to the image points (imgpoints) array to keep the chessboard corners. I also replicated the object points and append to 'objpoints' array to correspond to the found image points.
After this, I used the outputs 'objpoints' and 'imgpoints' to compute the camera calibration matrix and distortion coefficients needed for correcting distortion on images by using the opencv cv2.calibrateCamera() function.
To be sure about the success of the calibration, I tested it on one of the chessboard images to correct its distortion by using the cv2.undistort() function. The figure below shows the image before distortion correction and after correction.x

![alt text][image1]

####Apply a distortion correction to raw images

I demonstrated the distortion correction on one the test images. The result is shown below.

![alt text][image2]

###2.0 Image Thresholding
####Use of color transforms, and gradients to create a thresholded binary image.

<!-- ####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images) -->

Before applying image thresholding, I investigated how the image appears in various colour spaces to decide on which channels will be useful for this purpose. The figures below a test image shown in different colour spaces.

![alt text][image3]

![alt text][image4]

From observing and earlier trials, I realized the best way to detect lane lines is to use the colour information. Since lane lines are usually yellow and white, it will be effective to detect only white and yellow patches of road. I therefore decided to use colour thresholding. I also realized the RGB channel is more effective in detecting the white patches, since the luminosity is not a separate channel, and the HLS channel is effect for detecting the yellow colours, since only channel represents the Hue values. The following table shows the values I used to threshold white and yellow in the respective colour channels.

| Colour 		| Threshold Values   										| 
|:-------------:|:--------------------------------------------------------:	| 
| RGB White     | Lower = {100, 100, 190}, Upper = {255, 255, 255}       	| 
| RGB Yellow 	| Lower = {230, 180, 20}, Upper = {255, 255, 255} 			|
| HLS Yellow 	| Lower = {20, 100, 30}, Upper = {45, 200, 255}     		|


The code for thresholding is contained in the threshold_colours() function of the 6th cell in the pipeline_sessions.ipnyb file

See Images below for output of thresholding from each colour space
![alt text][image5]

![alt text][image6]

![alt text][image7]

###3. Perspective Transform
####Apply a perspective transform to rectify binary image ("birds-eye view")

<!-- Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image. -->

I loaded a test image with straight lines, to obtain points on the road plane. Points were manually picked from the straight road lines. I also chose destination points to transform to a birds eye view. This operation is in the Perspective Transform section (3.0)　of the pipeline_sessions.ipynb file.

See below, the points I chose for source and destination:

<!-- The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```
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

``` -->
<!-- This resulted in the following source and destination points: -->

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 270, 674      | 270, 674      | 
| 587, 455      | 270, 0      	|
| 694, 455     	| 1035, 0      	|
| 1035, 674     | 1035, 674     |

To verify the perspective transform, I drew line on points and transformed it to bird's eye view. Result is shown in figure below:

![alt text][image8]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

