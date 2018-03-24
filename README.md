

# Advanced Lane Finding Project

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.

* Apply a distortion correction to raw images.

* Use color transforms, gradients, etc., to create a thresholded binary image.

* Apply a perspective transform to rectify binary image ("birds-eye view").

* Detect lane pixels and fit to find the lane boundary.

* Determine the curvature of the lane and vehicle position with respect to center.

* Warp the detected lane boundaries back onto the original image.

* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: #	"Image References"
[image1]: ./examples/undistortedImage.png	"Undistorted"
[image2]: ./examples/undistorted_test_image.png	"Real Images"
[image3]: ./examples/binary_image.png	"Binary Images"
[image4]: ./examples/binary_warping.png	"Binary Warped Images"
[image5]: ./examples/lane_fitting.png	"Lane Fitting Images"
[image6]: ./examples/lineArea1.png	"Output Area1"
[image7]: ./examples/lineArea2.png	"Output Area2"
[video1]: ./project_video.mp4	"Video"
[]: 

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb" (or in lines # through # of the file called `some_file.py`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination (binary OR) of two binary images, the one created by sobelx and sobely operations and the one created by tresholding the HLS image. 



RGB binary thresholds (sobel):

threshx_min = 50
threshx_max = 100
threshy_min = 50
threshy_max = 100



HLS (s and v channel) tresholds: 

 s_thresh = (110, 255)
 v_thresh = (60, 255)



The example image shows the conversion from the real image over the two binary images to the result binary image (on the right)

```
original image, binaryImage(sobel), binaryImage(hls), ConcatenatedImage(bin1 OR bin2)
```

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

i used the warpImage() function to warp the binary image to a birdseye view. The src and destination points were defined arbitrary onto the road.

```python
srcPoints = np.float32([(220,720), (570,470), (720,470), (1110,720)]) 
destPoints = np.float32([(220,720), (220,0), (1110,0), (1110,720)])
```

I verified that my perspective transform was working as expected by drawing the warped image and verifying the results.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used the slidingWindows() function to select the right and left lane pixels (by a peak in a histogram) which are likely to correspond to the road markings by creating multiple windows to get a more discrete representation of the most likely road marking. These windows are then passed to the fitWindows() function, where a second order (n2) polynomial will be fitted into the pixels of these windows. The result of this function are the lanes itself, which consist of all their corresponding pixels, the fitting coefficients and other features. 

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated the curvature of the lanes in the calculateCurvature() function by fitting a circle to our polynomial as described in the course.

i calculated the center-offset between the car and the lanes in the centerOffset() function, which uses the fitted polynomials and the image center to calculate the diff of the respective lane sides. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.



![alt text][image6]

![alt text][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4])

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I did not implement any filtering algorithms, so that the fitting can jump unpredictably. The algorithm is also not very robust against changes in light levels. A plausibility check for our lane fittings could result in more desirable results as well. If the road is a S-curve we would need higher order polynomials to fit reliably, so a MSE over the fitting results could decide whether or not we want to fit with n=1, n=2, n=3 etc.
