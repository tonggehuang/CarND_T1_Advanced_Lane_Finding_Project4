
## Advanced Lane Detection

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

[image1]: ./output_images/undistort_effect.png "Undistorted1"
[image2]: ./output_images/undistort_test.png "Undistorted2"
[image3]: ./output_images/threshold_determination1.png "threshold_determination"
[image4]: ./output_images/perspective_trans.png "perspective_trans"
[image5]: ./output_images/hist.png "hist_show"
[image6]: ./output_images/sliding_window.png "sliding_window"
[image7]: ./output_images/previous_find.png "previous_find"
[image8]: ./output_images/project_back.png "project_back"
[image11]: ./output_images/chessboard_cornerfind.png "chessboard_cornerfind"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image11]

### Pipeline (single images)

#### 1. Distortion-corrected image

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![alt text][image2]

#### 2. Threshould Playground

I will first clip some sample frames from the 3 given video with differenct road type and narual environment to adjust threshold parameter effectively and robust. Also, I used a dynamic threhold turing based on the intensity of the grayscale histgram to deal the dark condition and shadow.

My approach contains the following steps:

1. Abs Sobel threshold on 'x' direction
2. Gradient direction: Avoid bridge edges which are more flat than the lane edges.
3. Grayscale intensity filter: Conditional change threshold for S, L, R, G channel based on the lightness. 
    1. HLS image color threshold for yellow lane: S channel for yellow lane detection, L channel for brightness adaptive.
    2. RGB image color threshold for white lane: R and G channel used for enhancing white lane detection.
4. ROI Mask
5. Remove small object: Denoise

![alt text][image3]

--- 

#### 3. Warped

```python
src = np.float32([[280,690],[570, 470],[700, 470],[1030, 690]])
dst = np.float32([[340,720],[320, 0],[960, 0],[1000, 720]])
```

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Lane Searching Window

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image6]

If the previous lane have been detected, we can skip the window search step.

![alt text][image7]

#### 5. Curverture Find and Project on Original Image

I used the value for meter per pixel from assumption.

```ym_per_pix = 30/720 # meters per pixel in y dimension```

```xm_per_pix = 3.7/700 # meters per pixel in x dimension```

Here is the final mixed screen image. I provide the mixed screen for checking the threshold image quality.

![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [Project video output](https://youtu.be/4_siFy8VVpg)

Here's a [Challenge video output](https://youtu.be/NUhzezyGM7o)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I have spent a lot of time on manually adjusting the threshold parameters in this project. I guess the threshold part is the bottleneck of my process. The video output quality is highly depended on how good the threshold parameters used for generating binary images. In my process, I used a dynamic threhold turing based on the intensity of the grayscale histgram to deal the dark condition and shadow. However, this is not enough for harder challenge video. For the improvement, I think the more meaningful color channels and filters showed be used in threshold turning step. Based on the road condtion, the adaptive threshold combination could be another good approach.  
