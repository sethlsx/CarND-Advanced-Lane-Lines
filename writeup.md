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

[image1]: ../CarND-Advanced-Lane-Lines/output_images/straight_lines1_out.jpg
[image2]: ../CarND-Advanced-Lane-Lines/output_images/straight_lines2_out.jpg
[image3]: ../CarND-Advanced-Lane-Lines/output_images/test1_out.jpg
[image4]: ../CarND-Advanced-Lane-Lines/output_images/test2_out.jpg
[image5]: ../CarND-Advanced-Lane-Lines/output_images/test3_out.jpg
[image6]: ../CarND-Advanced-Lane-Lines/output_images/test5_out.jpg
[image7]: ../CarND-Advanced-Lane-Lines/output_images/test6_out.jpg
[image8]: ../CarND-Advanced-Lane-Lines/output_images/test4_out.jpg
[video1]: ../CarND-Advanced-Lane-Lines/output_video/project_video_output2.mp4


**All the codes for this project is contained in the IPython notebook located in "./Advanced_Lane_lines_2nd_try.ipynb".**

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!



### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./Advanced_Lane_lines_2nd_try.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

This is the original image before undistortion.
![original_image][../CarND-Advanced-Lane-Lines/camera_cal/calibration2.jpg]
This is the undistort image.
![undistort_image][../CarND-Advanced-Lane-Lines/output_images/calibration2_out.jpg]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![undistort_straight_lines][../CarND-Advanced-Lane-Lines/output_images/straight_lines1_undist.jpg]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. I define multiple functions to perform this. These functions are in the code cell 5 through 8. 

First, I use the sobel function on x and y direction, the results are showed in the following images:

![gradx][../CarND-Advanced-Lane-Lines/step_by_step/test2_gradx.png]
![grady][../CarND-Advanced-Lane-Lines/step_by_step/test2_grady.png]

Then I apply the magnitude threshold.

![mag][../CarND-Advanced-Lane-Lines/step_by_step/test2_mag.png]

Then the direction threshold.

![dir][../CarND-Advanced-Lane-Lines/step_by_step/test2_dir.png]

Then s channel of hls.
![s_channel][../CarND-Advanced-Lane-Lines/step_by_step/test2_s_channel.png]

Then I combine them together.
![combined][../CarND-Advanced-Lane-Lines/step_by_step/test2_combined.png]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_trans()` in the code cell 9.  This function takes as inputs an image (`binary_img`). Using the canny and houghlinesp API from last project, I generated the src and dst points for the perspective transform.

When I was coding, I wrote mutiple plt.imshow() function to check the image transformation step by step to make sure the transformation didn't go wrong. Althoug I comment these lines out, they are still left in the code for future tuning.

Here is an example of the transformed image.

![warped][../CarND-Advanced-Lane-Lines/step_by_step/test2_warped.png]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I define functions in code cell 10 through 11 to perform this job. I used the sliding windows methods to identify the lines and fit second order polynomial to them.

Here in code cell 12 I define function to search for lines based on the lanes found in the prior frame. 

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in code cell 13 to 14.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

All the functions are combined in a function called pipelin in code cell 16. And I display all the output on the test images in the jupyter notebook. And here is one of them:

![straight_lines_1_out][../CarND-Advanced-Lane-Lines/output_images/straight_lines1_out.jpg]

And all the other images are stored here:'../CarND-Advanced-Lane-Lines/output_images/'

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](../CarND-Advanced-Lane-Lines/output_video/project_video_output2.mp4)

As I apply the codes to video, I rearrange the functions so that I can process the video frame by frame, and most importantly, process some frame based on the prior frame.

Here are some changes I made.

- I only apply the hough method to find lines to calculate the perspective transformation matrix on the first frame, and all the following frames are processed assuming the matrix stays the same through out the video. This will make the process much smoothier than before. In real life, the hough method could be called every few seconds to rectify the matrix.
- I call the search_around function on the frames following the frame in which lanes having been found.
- I smooth the lanes parameters between frames, which worked very well. So I also smooth the curverad.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

When I tuned my code, I found they were too much tuned according to the video or images to be dealt with. The part where the code detect the points for the perspective transformation would probably fail when the vehicle face the road from a different angle. If I could learn how to detect or decide the src and dst points for perpective tranformation automatically in a different, more robust way, the codes would be much better.
