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

[image1]: ./output_images/undistort_chessboard.jpg "Undistorted"
[image2]: ./output_images/undistort_test1.jpg "Road Transformed"
[image3]: ./output_images/binary_warped.jpg "Binary Example"
[image4]: ./output_images/warpPerspective_points.jpg "Warp Example"
[image5]: ./output_images/lane_color_warp.jpg "Fit Visual"
[image6]: ./output_images/result.jpg "Output"
[video1]: ./output_images/project_video_lanes.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "./examples/example.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

Just the same as undistorting the chessboard, an image from the vehicle camera is read in and the distortion correction is applied using the `cv2.undistort` function.

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image. (I actually performed a perspective transform before the color/gradient transforms, so this question comes before question #2.)

The code for my perspective transform appears in a function called process_image in cell 11 of my jupyter notebook. The function takes as inputs an image (`image`), undistorts the image as described above, and warps the perspective with the function `cv2.warpPerspective()`.  I chose to hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 60, img_size[1] / 2 + 100],
    [(img_size[0] / 6), img_size[1]],
    [(img_size[0] * 5 / 6) + 50, img_size[1]],
    [(img_size[0] / 2 + 60), img_size[1] / 2 + 100]])
dst = np.float32(
    [[((img_size[0] / 4)), 200],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [((img_size[0] * 3 / 4)), 200]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 580, 460      | 320, 200      | 
| 213, 720      | 320, 720      |
| 1117, 720     | 960, 720      |
| 700, 460      | 960, 200      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps in cell 7 in the jupyter notebook).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

As seen in cell 10 of my jupyter notebook, I used two techniques to finding lane linds, depending on if it is the first time finding lanes or if lanes have been found before.

##### First time:
Take a histogram of the bottom half of the image. Then find the peak of the left and right halves of the histogram as these will be the starting point for the left and right lines. Next, define the sliding windows: I chose 9 windows with a window height of <heightOfImage/9>, 80 pixels in this case. Then, identify the x and y positions of all nonzero pixels in the image. Then step through the windows one by one, and find a search area with the most lane line pixels in the windows, identified by the margins. Finally, get the locations of those pixels and fit a 2nd order polynomial to them.

##### Every other time
Search within a margin, in this case 100 pixels, of the last lane line found, and find areas of the image with the most lane line pixels. Finally, fit a 2nd order polynomial to those pixel coordinates.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Curvature is calculated in cell 9 of my jupyter notebook. First, meters per pixel in both the x and y direction are defined. Then new polynomials are fit to the left and right lane line points in the world space. Finally, left and right curve radius is calculated using the polynomial coefficients.

Position of the vehicle with respect to center is calculated in lines 78-82 in cell 11 of my jupyter notebook. First, the image center in the x direction is found, representing the center of the camera, which is placed in the center of the car. Then the offset is calculated as follows: `centerx - center point between the lower x intercepts of left and right lane lines`. This value is then multiplied by the calculated meters per pixel value to get a value in meters. If this value is negative, the vehicle is left of center, and if it is positive, the vehicle is right of center.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines 93-129 in cell 11 of my jupyter notebook. Here is an example of my result on the first image of the project video:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result][video1]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I mostly faced issues with printing the lane overlay on top of the video, which stemmed from not getting the correct reference points for the perspective translation. I wanted to extend the overlay as far as I could, rather than cutting off halfway up the image, so I played around with ways of getting as much of the road in the birds-eye-view image as I could to better draw an overlay, as well as keeping the entire lane in the frame laterally, even with tighter turns.

If I were to continue this project, I'd focus on faster implementation of the lane finding algorithm as well as smoothing the output,  detecting bad frames, and applying a mask to the binary image to only limit the view to the road ahead.
