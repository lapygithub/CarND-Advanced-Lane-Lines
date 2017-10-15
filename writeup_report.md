## ** P3 - Advanced Lane Finding Project **

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

[image1]: ./examples/undistort_output_mrl.png "Undistorted"
[image2]: ./examples/test1_undist_mrl.png "Road Transformed"
[image3]: ./examples/binary_combo_example_mrl.png "Binary Example"
[image4]: ./examples/warped_straight_lines_mrl.png "Warp Example"
[image5]: ./examples/warped_straight_lines2_mrl.png "Warp Example 2"
[image6]: ./examples/polyfit_output_mrl.png "Poly-Fit Visual"
[image7]: ./examples/example_frame_output_mrl.png "Frame Drawn Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./p3.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  In the second code cell of the IPython notebook, the test image is undistorted via a the function `cal_undistort()` which will be called later in the code.
From cv2 examples, `cal_undistort()` implements the camera matrix optimization function `cv2.getOptimalNewCameraMatrix()` ([Link] (http://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_calib3d/py_calibration/py_calibration.html)) before it applys distortion correction to the test image using the `cv2.undistort()` function.  Here is an example of the result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one showing original and undistorted images (See P3.ipynb third code cell):
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps in P3.ipynb fourth code cell).  Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the 5th code cell of the IPython notebook).  The `corners_unwarp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 30), img_size[1]],
    [(img_size[0] * 5 / 6) + 70, img_size[1]],
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
| 183, 720      | 320, 720      |
| 1137, 720     | 960, 720      |
| 720, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In the section titled "7) Polynomial Fit Find Function", the function polyfit_find() is defined to find the lane lines in the input image result of the undistorted, unwarped, top down, binary image.  Right and left lines are searched using a window margin left and right of center making use of histogram maximum peaks.  Nine horizontal line sections are used.

My implementation searches for lane lines on each frame, while a clear optimization would be to make use of frame to frame cohearance to avoid the searching on every frame.

A polynomial is then fit to the centroid of each window and used to calculate the frame by frame video overlay polygon.

An image view of the processing, including the histogram details:

![alt text][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The functions to calculate lane curvature (get_curvature()) and offset from center (get_dist()), can be found in P3.ipynb and the cell titled "8) Measuring Curvature".

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented plotting back to the road directly in the `process_image2()` function in the step labeled `10) Putting It All Together`. cv2.fillPoly() is used to draw the polgon created from the returned values of polyfit_find().
Here is an example of my result on a test image:

![alt text][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/project3_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.

My approach was to implement the course outline materials in sections making use of Jupyter cells to confirm results.  The following cells need to be run in order: `1) Camera Calibration`, `2) Undistort Function`, `3) Threshold Gradient Pipeline`, `4) Unwarp Function`, `7) Polynomial Fit Find Function`, `8) Measure Curvature`, `9) Tracking Class`, `10) Putting It All Together` and finally `11) Process Video`.

After debugging all of the main components, the best improvement to the final video mask calculation was the implementation of 5 frame averaging and the rejection of bad polygon fits (lane lines not found).  I used Python collections.deque() to store and remove the five polyfits to average.  See the cell labeled `9) Tracking Class` which also stores lane line information over the video processing.

I had major delays intially getting the video to process as the overlay mask was not changing frame to frame and then wasn't being calculated correctly.  Eventually, I figured out that there was an issue with Jupyter's cell to cell global variables.

The video shows that the lane lines overlays are not perfect with little edge wiggles most likely due to shadows.  This would indicate that more work could be done augmenting the threshold gradient algorithm possibly using different color channels to pop the lane lines even more around shadows.  The current algorithm uses Sobel x on the HLS lightness channel.
The overall pipeline could also be made more efficient frame to frame as outlined in the course notes with the unused function `polyfit_next()` in the cell labeled `7a) Polynomial Fit Next Frame`.

This pipeline, while it works for the current video, will have issues if the first frame doesn't have clear lane lines.  Curvature may also have issues with hills as the perspective view is assumed to be flat.
