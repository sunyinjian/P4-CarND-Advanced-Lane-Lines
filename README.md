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

[calibration_image]: ./output_images/calibration.png "Calibration"
[undistorted_image]: ./output_images/undistorted_image.png "Undistorted Image"
[gradient_threshold]: ./output_images/gradient_threshold.png "Gradient Threshold"
[color_threshold]: ./output_images/color_threshold.png "Color Threshold"
[overall_combined]: ./output_images/overall_combined.png "Overall Combined"
[warp_image]: ./output_images/warped_img.jpeg "Warped Image"
[all_image]: ./output_images/all_img.png "All Image"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 'P4.ipynb' (line 2 to 28 In [2] ). 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. Thus, `obj_points` is just a replicated array of coordinates, and `obj_points` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. `img_points` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `obj_points` and `img_points` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][calibration_image]

The camera calibration result and perspective transform matrix were saved as a pickle file. 

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

A raw image was loaded by `cv2.imread` function. Then the raw image was undistorted by `cv2.undistort` function. The resultant image is:

![alt text][undistorted_image]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image.  

The `gradient_thresh` function in [10] was used to generate a binary image. Here is an image combining thresholds of sobelx and direction:

![alt text][gradient_threshold]

The `color_thresh` function in [11] was used to generate a binary image. I combined R & G channels of RGB to detect white lines well. Then I combined L & S channels of HLS to detect bright yellow and white lines and to avoid shadows. The image using color thresholds is:

![alt text][color_threshold]

The `threshold_pipeline` function in [12] was to combine image using gradients threshold and color transforms. Here is an image:

![alt text][overall_combined]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The perspective transform matrix was calculated and saved in a pickle file (line 2 to 12 in [5])
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 570, 470      | 320, 0        | 
| 220, 720      | 320, 720      |
| 1110, 720     | 920, 720      |
| 720, 470      | 920, 0        |

To generate perspective transform of an image, function `warp_image` in [4] was used to warp the image. The warped image is:

![alt text][warp_image]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

A histogram of pixels was built to identify lane-line pixels from bottom to top. The corresponding codes are in `first_sliding_window` function in [17] and `second_sliding_window` function in [18]. Then the pixels are fitted by polynomial.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines 8 to 67 in the `second_sliding_window` function in [18]. I used the function `collections.deque(maxlen = 10)` to store the last 10 frames. If the pipeline encountered a bad frame, I used the average of the past 10 frames. I smoothed the lane weighting the current frame and the mean of the last 10 frames. 
The method of identifing bad lines refers to Vivek Yadav (https://chatbotslife.com/robust-lane-finding-using-advanced-computer-vision-techniques-46875bb3c8aa)

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines 5 through 30 in my code in `combine_images` function in [19]. The thresholed image and the perspective transformed image are plotted in one image. Here is an example of my result on a test image:

![alt text][all_image]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_result.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

When the light condition changes in challenge video, the pipeline may fail. 
The solutions are sensitive to the parameters. I tried to use different combination of channels including R, G, L, S and gradient thresholds.
The points used to calculate perspective transform matrix is not accurate. Sometimes the lanes went outside the region of interest. The sccurate calibration of the perspective transform should be applied. 
The deep learning techniques could be used for robust lane finding. 

