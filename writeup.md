## Writeup

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

[undist_1]: ./res/undistort_1.png "Undistorted"
[undist_2]: ./res/undistort_2.png "Undistorted"
[undist_3]: ./res/undistort_3.png "Undistorted"
[undist_4]: ./res/undistort_4.png "Undistorted"
[undistort]: ./res/undistort.png "Undistorted"
[threshold]: ./res/threshold.png "Threshold"
[perspective_transform]: res/perspective_transform.png "Perspective Transform"
[moving_window]: res/moving_window.png "Moving Window"
[birds_eye]: ./res/birds_eye.png "Birds Eye"
[filled_poly]: ./res/filled_poly.png "Filled"
[overlay]: ./res/overlay.png "Overlay"
[final]: ./res/final.png "Final"
[project_video]: ./output_videos/project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.

You're reading it!

### Camera Calibration

The code for this step is contained in the first code cell of the IPython notebook located in "advanced_lane_lines.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function [1].  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][undist_1]
![alt text][undist_2]
![alt text][undist_3]
![alt text][undist_4]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][undistort]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I evaluated the combinations of gradient direction, magnitude and color threholds. The combined thresholds of RGB/HLS and gradient magnitude outperforms other in the test images. (code cell **4** of `advanced_lane_lines.ipynb`).  Here's an example of my output for this step.

![alt text][threshold]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes functions called `perspective_transform_to_from_birds_eye()` and `warp()`, which appears in code block **5** of `advanced_lane_lines.py`.  The `perspective_transform_to_from_birds_eye()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
    # Grab the image shape
    img_size = (img.shape[1], img.shape[0])

    src = np.float32([(590, 460), (709, 460), (1093, 719), (230, 719)])
    # For destination points, I'm arbitrarily choosing some points to be
    # a nice fit for displaying our warped result
    # again, not exact, but close enough for our purposes
    dst = np.float32([[300, 0], [980, 0], [980, img_size[1]-1], [300, img_size[1]-1]])
```

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 590, 460      | 300, 0        |
| 709, 460      | 980, 0        |
| 1093, 719     | 980, 719      |
| 230, 719      | 300, 719      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][perspective_transform]

With the calculated perspective transform matrix, I was able to convert the threshold binary into birds' eye view.
![alt text][birds_eye]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I searched for the lane line pixels using moving windows and fit them with a 2nd order polynomial:

In the following example:
1. Green rectangles are the moving windows to find pixels for left and right lane lines.
2. Red points are pixels found for left lane while blue points belong to the right lane.
3. Yellow lines are the polynominals fit to both lanes.

![alt text][moving_window]

Searching for the lane-line pixels is done from bottm to top.
- Firstly a histogram with two peaks is calculated to locate the bottom of both lanes.
- Secondly a window of 200x80 is moved up along the lane lines to locate the next segment. In every substep, mean of the horizontal positions is calculated on pixels within the current window. This mean position will be used as horizontal center of the next window.
- Finally pixels in these windows are used to fit two polynomilas - one for the left lane line and one for the right lane line.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in code block **7** of `advanced_lane_lines.py`. Algorithm introduced at Udacity classroom [2] was modified to return radius as signed numbers thus we can tell whether the curve is turning left or right. (Negative -> left, positive -> right)

In code block **14** of `advanced_lane_lines.py`, a simple IIR filter was also used to make the radius output more smooth.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code block **8**, **9**, and **10** of `advanced_lane_lines.py`.  Here is an example of my result on a test image:

![alt text][filled_poly]
![alt text][final]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

To make the piple work more efficently for video stream, the following changes have been made:
1. When looking for lane line pixels in **Step 4**, there is no need to run `histogram()` from the bottom. A better approach is to reuse the polynominal from last frame. In the new frame, we firstly find all the pixels within a horizontal margin around the last poly line. Then these pixels will be used to fit the new polynominal. The old approach will still be applied if a new polynominal couldn't be fit.
1. A sanity check is performed upon left and right radius of curvature. If they are going different directions or the difference is too big, `histogram()` will be applied to restart the moving windows from bottom of image.
1. An average filter is applied to the last five polynominal fits. The filled region becomes less woblly with the averaged coefficients.
---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

### References
1. https://docs.opencv.org/3.1.0/dc/dbb/tutorial_py_calibration.html
2. https://classroom.udacity.com/nanodegrees/nd013/parts/fbf77062-5703-404e-b60c-95b78b2f3f9e/modules/2b62a1c3-e151-4a0e-b6b6-e424fa46ceab/lessons/096009a1-3d76-4290-92f3-055961019d5e/concepts/2f928913-21f6-4611-9055-01744acc344f