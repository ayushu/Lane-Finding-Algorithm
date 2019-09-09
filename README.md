# Lane Finding Algorithm

This project was done as part of the [Udacity Self-Driving Car Nanodegree](http://www.udacity.com/drive)

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

[image1]: ./writeup_images/undistorted.png "Undistorted"
[image2]: ./test_images/test1.jpg "Test Image Distorted"
[image3]: ./writeup_images/test1_undistorted.jpg "Test Image Undistorted"
[image4]: ./writeup_images/binary_image.png "Binary Image"
[image5]: ./writeup_images/birds_eye.png "Bird's Eye View"
[image6]: ./writeup_images/histogram.png "Fit Visual"
[image7]: ./writeup_images/polynomial.png "Fitted Polynomial"
[image8]: ./writeup_images/detected_lanes.png "Detected Lane with Metrics"
[video1]: ./videos_output/project_video.mp4 "Video"

---

## Camera Calibration

### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for the camera calibration and distortion correction step are contained in the cell of the IPython notebook located in "./P2.ipynb" under the heading 'Camera Calibration and Distortion Correction'.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners. Since, the chessboard is on a flat surface, it can be assumed that the xy-plane is fixed at z = 0 for all objects points across all calibration images. `objectp` is just an array of the coordinates while `objpoints` is appended with a copy of `objectp` every time the chessboard corners are detected in a test image.

Finally, the `cv2.CalibrateCamera()` function was used to compute the camera calibration and distortion coefficients. These values were used along with the `cv2.undistort()` function to undistort the image. This was the result:

![alt text][image1]

## Pipeline

### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will apply the distortion correction to the following test image:

![alt text][image2]

Below is the the same image with the distortion correction applied:

![alt text][image3]

### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

In order to create a thresholded binary image of the road, I used a directional gradient, a gradient magnitude, a gradient direction, and a color threshold. These can be found in cells 5 - 10 of the IPython notebook. The following are the functions used in my pipeline:

* Directional Gradient: `abs_sobel_thresh()`
* Gradient Magnitude: `mag_thresh()`
* Gradient Direction: `dir_threshold()`
* Color Threshold: `hls_select`

Finally, I combined the color and gradient thresholds using the `thresh_combine()` function to create the binary image.

Below is the binary image created next to the undistorted image used as the input:

![alt text][image4]

### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

My code for the perspective transform uses the `birds_eye()` function, which is in cell 11 of the IPython notebook. The `birds_eye()` function takes as inputs an image (`img`), as well as source (`src_coordinates`) and destination (`dst_coordinates`) points.  I chose the hardcode the source and destination points as follows:

```python
src_coordinates = np.float32(
    [[280, 700], 
    [595, 460], 
    [725, 460], 
    [1125, 700]])
dst_coordinates = np.float32(
    [[250, 720], 
    [250, 0], 
    [1065, 0], 
    [1065, 720]]) 
```

For clarity, the source and destination points are:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 280, 700      | 250, 720      | 
| 595, 460      | 250, 0        |
| 725, 460      | 1065, 0       |
| 1125, 700     | 1065, 720     |

I verified that my perspective transform was working as expected by drawing the `src_coordinates` and `dst_coordinates` points onto a test image and its bird's eye counterpart to verify that the lines appear parallel in the warped image.

![alt text][image5]

### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

There were several steps performed in the pipeline to identify the lane-line pizels and then fit their positions with a polynomial. The first step was the use a histogram of the bottom half of the binary image to identify two peaks, which would relate to the lane-line. This can be found in the 13th cell of the IPython notebook. The following is the histogram for the test image seen previously in this write-up:

![alt text][image6]

Then, to detect the lines, the lane is split down the middle to separate the left and right lane-line. The sliding window and the search from prior techniques were used to identify individual lane-lines. Found in cells 15 - 17, the functions used are named as follows:

* `find_lane_pixels()`
* `fit_polynomial()`
* `fit_poly()`
* `search_around_poly()`

The sliding window technique identifies the most probable lane coordinates in a small window and this slides vertically through the image. Using the coordinates calculated using the above techniques, a second-order polynomial was calculated for each lane-line. The second-order polynomial was calculated using the `np.polyfit` function. The identified lane-lines along with the fitted polynomials are shown below:

![alt text][image7]


### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radii of curvature and the position of the vehicle with respect to center were calculated using the `measure_metrics()` function in the 19th cell of the IPython notebook. It uses the midpoint of the image and compares it with the point in between the two lane-lines to determine the offset value. The polynomial coeeficients for each lane-line were used to calculate the lane curvature radius. All three values were then converted to metres using a pixel-to-m conversion rate for the x and y directions, respectively.

### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

In the final step of the pipeline, found in cells 21 - 23, the lane-line points are used to mask the identified lane onto the lane using the `create_lanes()` function. However, the identified lane needed to be unwarped back to the original perspective, and then needed to be weighted such that the lane itself was still visible under the lane highlight. The left lane-line was marked in red and the right lane-line was marked in blue. Finally, the `write_metrics()` function is used to write the previously calculated metrics onto the image. Below is the final output with the lane marking and the metrics:

![alt text][image8]

---

## Pipeline (video)

### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./videos_output/project_video.mp4)

---

## Discussion

### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

While the pipeline worked successfully on the project video, there are still many improvements that can be made. 

* The first one is the efficiency of the pipeline. It took around an hour to process the 50 second video. Certain functions are called repeatedly within other functions and this could be made more efficient by having a more robust `process_image()` function.
* A mode or median lane distance can be used to ensure that all subsequent lane width measurements are consistent to make sure that there are no anomalies.
* Finally, there might be a risk of the pipeline not accurately recognizing the lane in shadowy conditions or similar, and this would also be an issue that would need to be fized in future iterations of the project.

## License
[MIT](./LICENSE)