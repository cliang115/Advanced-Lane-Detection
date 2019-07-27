## Writeup 

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

[image1]: ./output_images/Undistorted_chessboard.png "Undistorted"
[image2]: ./output_images/test_image.png "Road Transformed"
[image3]: ./output_images/thresholded_image.png "Binary Example"
[image4]: ./output_images/warped_image.png "Warp Example"
[image5]: ./output_images/polynomial_lines.png "Fit Visual"
[image6]: ./output_images/rewarped_image "Output"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in the first code cell of "./examples/example.ipynb".

I used the given chessboard images for camera calibration. I detected the corners with the cv2.findChessboardCorners function. If I found the corners, I appended the object points and corners to `objpoints` and `imgpoints`, respectively. I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

I applied this distortion correction to the '../test_images/test1.jpg' using the `cv2.undistort()` function and obtained this result: 

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I performed color transforms and used a combination of color and gradient thresholds to generate a binary image (thresholding steps in the "thresholding()" fucntion in the third code cell in "./examples/example.ipynb").  Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which appears in the 4th code cell of the IPython notebook).  The `warp()` function takes as inputs an image (`img`).  I chose the hardcode the source (`src`) and destination (`dst`) points in the following manner:

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

Here is an example of my output after perspective transform:(The lane lines seem to be straight)

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

For the first image in a video, I identified lane-line pixels with the 'sliding window' method. I take a histogram along all the columns in the lower half of the image. The two most prominent peaks in this histogram will be good indicators of the x-position of the base of the lane lines. From that point, I used a sliding window, placed around the line centers, to find and follow the lines up to the top of the frame. I split the histogram into two sides, one for each lane line. Then I iterate through nwindows to track curvature. The code for this part is in the "find_lane_pixels" function in sixth code cell of the notebook.

For the following frames in a video, I identified lane-line pixels with the 'search from prior' method. I take the polynomial functions I fit before and grab only those pixels with x-values that are +/- margin from the polynomial lines. The code for this part is in the "search_around_poly" function in the 8th code cell of the notebook.

Once I identified lane-line pixels, I fit their posisions with np.polyfit function. The code for this part is in the "fit_polynomial" function in the sixth code cell of the notebook.

Here is an example of my polynomial lines:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I used the radius of curvature equation from the course and first converted the coeffients "A" and "B" to meters. I used the average of the left and right lane radius of curvatures.I calculated the position of the vehicle with respect to center by subtracting lane_center which is the middle of the two lane bases and view_center which is the middle of the image. I converted the position of vehicle to meters.
I did this in the first 11 lines and last five lines of the function "rewarp" in the seventh code cell of the notebook.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines 14 through 29 in my code in the function `rewarp()` of the notebook.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The biggest problems I faced in my implementation of this project is that the pipeline cannot find the accurate polynomial lines when the background of the road is grey(color is lighter). I tried to twist the parameters of color/gradient thresholds as well as the margins for looking for lane line pixels. By changing these parameters, I could identify lane area more accurately. But sometimes I still cannot detect the exact positions of the lane lines.

The pipeline will likely fail if the lane is more curvy. Discovering lane lines using the 'sliding windows' approach instead of 'search from prior' seems to help, but still fails if the turn is very sharp. In that case, cameras with adjusting angles will likely help with the perspective transform step. 

The pipeline may also fail if the ground color varies within the lane. Better thresholding method will help to cope with this situation.

