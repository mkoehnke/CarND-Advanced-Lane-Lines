# Writeup

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

[img1]: ./output_images/chessboard_output.jpg "Chessboard"
[img2]: ./output_images/undistort_output.jpg "Undistorted"
[img3]: ./output_images/sobel_x_output.jpg "Sobel X"
[img4]: ./output_images/sobel_y_output.jpg "Sobel Y"
[img5]: ./output_images/magnitude_output.jpg "Magnitude"
[img6]: ./output_images/direction_output.jpg "Direction"
[img7]: ./output_images/hls_s_output.jpg "HLS S-Channel"
[img8]: ./output_images/hls_l_output.jpg "HLS L-Channel"
[img9]: ./output_images/lab_b_output.jpg "LAB B-Channel"
[img10]: ./output_images/combined_output.jpg "Combined"
[img11]: ./output_images/source_points_output.jpg "Source Points"
[img12]: ./output_images/warped_output.jpg "Warped"
[img13]: ./output_images/histogram_output.jpg "Histogram"
[img14]: ./output_images/polynomial_fit_output.jpg "Polynomial Fit"
[img15]: ./output_images/pipeline_output.jpg "Pipeline Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it! In order to follow the code samples for this project, please open the Jupyter Notebook I created:

```
jupyter notebook Advanced-Lane_Lines.ipynb
```

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][img1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The code for this step is contained in the fourth code cell of the IPython notebook. 

The following example shows the image `straight_lines1.jpg` from the `examples` folder, that has been distortion corrected:
![alt text][img2]


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The code for this step is contained in the 5-13 code cells of the IPython notebook. 

In order to create a thresholded binary image, I tried all learned methods like Sobel Operator X/Y Direction, Magnitude of the Gradient, Direction of the Gradient, Color Transformation to HLS + using S/L Channels. In addition, I tried Color Transformation to the LAB Color Space + using the B-Channel. Here are examples of my output for this step:

![alt text][img3]
![alt text][img4]
![alt text][img5]
![alt text][img6]
![alt text][img7]
![alt text][img8]
![alt text][img9]

I ended up using a combination of B-Channel (LAB) and L-Channel (HLS) because I found them to work the best with various light conditions:

![alt text][img10]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for this step is contained in the code cells 14-15 of the IPython notebook.

The code for my perspective transform includes a function called `warp()`, which takes as input an image and uses the following hardcoded source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 800, 520      | 1050, 520        | 
| 1050, 680      | 1050, 680      |
| 270, 680     | 270, 680      |
| 490, 520      | 270, 520        |

To help me find the correct source points, I plotted 'dots' onto the source image:
![alt text][img11]

The following example shows the before/after images of the perspective transform:

![alt text][img12]


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for this step is contained in the code cells 16-18 of the IPython notebook.

First I took a histogram of the binary image along all the columns in the lower half of the image like this:

![alt text][img13]

Secondly, I identified the most prominent peaks and used them as a starting point for where to search for the lines. From that point, I used the provided implementation of a sliding window, placed around the line centers, to find and follow the lines up to the top of the frame. Finally, the `polyfit()` function was used to fit a second order polynomial to each identified lane line:

![alt text][img14]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for this step is contained in the code cells 19-20 of the IPython notebook.

First, I defined the (provided) conversions for x,y from pixel space to meters. Then I identified the x,y positions of all nonzero pixels in the image, extracted the left/right line pixel positions and used `polyfit()` to fit polynomials to x,y in world space. Afterwards, I calculated the radii of curvature by using the provided sample implementation. In order to calculate the position of the vehicle with respect to center, I first calculated the absolute car position (image width/2) and the lane center position using the identified left/right lanes. Then I determined the center value by using the following calculation: `(car_position - lane_center_position) * x_meters_per_pixel`.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code for this step is contained in the code cells 21-22 of the IPython notebook.

I implemented the function `pipeline(img)` that returns an image (that has been processed by all the steps mentioned above) with the identified lane overlay.  Here is an example of my result on a test image:

![alt text][img15]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4).

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

##### Problems / Issues

During the implementation / testing phase of this project, I faced a lot of issues that were mostly related to difficult light conditions. Many methods of creating a binary image (Sobel, etc) where not able to perform well under these conditions. Only the combination of LAB (B-Channel) and HLS (L-Channel) performed well enough for `project_video.mp4`.

##### Where will it fail?

My current implemention will probably fail under conditions like, night time, Snow, Heavy Rain. In addition things like absence of lane markings, construction work, lane merges and Cars/Trucks in front of the car will probably cause lane identification issues.

##### How to improve?

Currently, my implementation doesn't make use of the function `find_hot_pixels_with_previous_fit()` which avoids doing a blind search for lane lines in each frame. Instead, it will search in a margin around the previous identified lines.

In terms of absence of lane markings, more sophisticated algorithms (like deep learning) might help to predict where lane positions should be.