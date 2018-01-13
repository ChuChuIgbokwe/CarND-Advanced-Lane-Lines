## Writeup Template

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

[image1]: ./output_images/undistorted.png "Undistorted"
[image2]: ./output_images/undistorted_2.png "Undistorted"
[image3]: ./output_images/pipeline_result.png "Binary Example"
[image4]: ./output_images/binary_warped.png "Warp Example"
[image5]: .//output_images/polyfit.png "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[image7]: ./output_images/projected_back.png "Result projected back onto the road"

[video1]: ./result_1.mp4 "Video"


### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/ChuChuIgbokwe/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

In Code Cell 2 of this notebook, I counted the number of inner corners on the chessboards
I read in the chessboard images using python's glob library
each image is converted to grayscale and passed to OpenCV's findChessboardCorners function which outputs the corners of each innner square in the chessboard.

Two lists are intitialized, objpoints and imgpoints. 
* objpoints will be appended with the (x, y, z) coordinates of the chessboard corners in the world
* imgpoints will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

The objpoints and imgpoints are passed into OpenCV's cv2.calibrateCamera() function, which computes the camera calibration and distortion coefficients.

I then use these coefficients and undistort a test image using OpenCV's cv2.undistort() function. The resaults can be seen in cell 3 or below

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

* I used a combination of color and gradient thresholds to generate a binary image. 
* In the cell above I defined helper functions for the gradiens and the magnituide of the gradient. 
* In the cell below I converted to HLS and computed the gradient and gradient magnitude for the S and L channel. 
* I also converted the image to HSV and I thresholded for yellow and white
* I combined my masked HSL and HSV image to create a binary image.
The results applied to the test images  can be seen 2 cells below

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform`, which appears in the cell below.  
The `perspective_transform` function takes as inputs an image (`img`), and returns a perspective transformation matrix and a warped image while calling the cv2.getPerspectiveTransform() and cv2.warpPerspective() functions respectively .  I chose the hardcode the source and destination points in the following manner:

```python
    top_left = (580,460)
    top_right = (707,460)
    bottom_left = (210,720)
    bottom_right = (1110,720)
    src = np.float32([[bottom_left, top_left, top_right, bottom_right]])

    left_bound, right_bound = (360, 970)
    top_left = (left_bound,0)
    top_right = (right_bound,0)
    bottom_left = (left_bound,720)
    bottom_right = (right_bound,720)
    dst = np.float32([[bottom_left, top_left, top_right, bottom_right]])

```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 360, 0        | 
| 707,460       | 970, 0      |
| 210,720       | 360, 720      |
| 1110,720      | 970, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

An example can be seen 2 cells below.



![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

* I create a binary image of the image then warp it to a birds eye-view.
* I take a histogram of this image and I use the two peak sin the histogram to find lines in the image
* I implemented a sliding window polytfit which works by slicing the image into windows of the same size and mark the windows that achieve a certain number of white pixels using the histogram. Selected windows are used in orde to fit a  second degree polynomial of the  that will be used to draw the lane.
* I implemented this in the cell below

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.
This is implemented in the cell below in the curvature function.
The pixels are scaled to meters and, A second order polynomial is then fit to the pixels using the equation ```curve_radius = ((1 + (2*fit_cr[0]*y_eval*ym_per_pix + fit_cr[1])**2)**1.5) / np.absolute(2*fit_cr[0])```



#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./result_1.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

* My pipeline could benefit from a more robust lane detection pipline, implementing a sanity check, look-ahead filter, reset and smoothing 
* Another imporvement would be standardizing images by resizing and equalizing the histogram to making it perform better in different lighting conditions would be another improvement
* I would also like to play around with more colour schemes and using a regression, RANSAC or deep learning approach to detect the lanes and comparing the results 
