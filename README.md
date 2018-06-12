## Advanced Lane Finding Project

[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)
---

The codes for this project is in the file [Advanced_Lane_Finding.ipynb](./Advanced_Lane_Finding.ipynb).

The output project video can be found here [project_video_output.mp4](./output_video/project_video_output.mpt). 

All output images is in the directory here [Output Images](./output_images).

---
**The goals / steps of this project are the following:**

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/image1.png
[image2]: ./output_images/image2.png 
[image3]: ./output_images/image3.png
[image4]: ./output_images/image4.png
[image5]: ./output_images/image5.png 
[image6]: ./output_images/image6.png
[image7]: ./output_images/image7.png
[image8]: ./output_images/image8.png
[image9]: ./output_images/image9.png
[image10]: ./output_images/image10.png
[video1]: ./output_videos/project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---



### Camera Calibration and Distortion Correction


The code for this step is contained in the Part 1 of the [IPython notebook](./Advanced_Lane_Finding.ipynb).  

I start by preparing "object points" and "image points" for each calibration image in the "camera_cal/" folder. I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]


### Pipeline (single images)

#### 1. Example of a distortion-corrected image.

In Part1, I saved `objpoints` and `imgpoints` lists to a pickle file. I also wrote a distortion-corretion function which based on these camera calibration results as "calUndistort".
Here is an example of a distortion-corrected image:
![alt text][image2]


#### 2. Thresholding

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at **Part 2**).  
In the combination of color and gradient thresholds, I chose the `and` result between L and S in HLS, R in RGB, and `and` result between sobel x and direction of the gradient. Then I used `or` result of all three of them as the final threshold. 

Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image6]

#### 3. Perspective Transform

The code for my perspective transform is the function called "image_processing(img)" in the **Part 3** of the Jupyter Notebook.  I chose the hardcode the source and destination points for the perspective transform in the following manner:


| Source        | Destination   | 
|:-------------:|:-------------:| 
| 597, 470      | 260, 1        | 
| 275, 680      | 260, 720      |
| 1030, 680     | 900, 720      |
| 683, 470      | 900, 1        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image7]

#### 4. Lane Line Identification

Here is the image showing the results of the histogram:

![alt text][image8]

I first used the sliding window method to identify the position of two lane lines. I seperated the image to 12 levels and in each level, I tried to find all qualified pixels in the area which was first identified by the histogram. Then, based on all the qualified pixels, I could found the best fit line for both of the two lanes. The left image below shows the result.

The function of the sliding window method is only for first image in the video. Once we know where the lane lines are, we can skip the sliding window part, but instead we can just search in a margin around the previous line position. The right image below shows this result.
![alt text][image9]

#### 5. Radius of curvature and vehicle position

The pixel values of the lane are scaled into meters using the scaling factors defined as follows:

```python
ym_per_pix = 30/720 # meters per pixel in y dimension
xm_per_pix = 3.7/660 # meters per pixel in x dimension
```

The equations for finding the radius of curvature are:

```python
    left_curverad = ((1 + (2*left_fit[0]*y_eval*ym_per_pix + left_fit[1])**2)**1.5) / np.absolute(2*left_fit[0])
    right_curverad = ((1 + (2*right_fit[0]*y_eval*ym_per_pix + right_fit[1])**2)**1.5) / np.absolute(2*right_fit[0])
```
"left_fit" and "right_fit" are the polynomial coefficients lists, and y_eval is the y axis position where we want to find the radius of the curvatures.

For the vehicle offset position, I used both undistorted image and top-view image, which is perspective transformed from undistorted image. I first calculated the ratio of horizontal pixel of the two images. Then, I converted the car center position which is in the undistorted image to the top-view image. In the last, I calculated the difference between lane lines center and converted car center and mutiplied the pixel-meter scaling factor to the estimate offset result.


#### 6. Example of Final result image 

The summary of my whole pipeline is showed here: 

**Distortion correction → Thresholding → Perspective Transform → Lane Detection → Curvature and Offset Calculation**

Here is an example image which has the lane lines identified and radius of curvature calculated:

![alt text][image10]

---

### Pipeline (video)


Here's a [link to my video result](./output_videos/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline I used works well in finding the lane lines in the project video. But it does not work very well for both the challenge_video and harder_challenge_video. It always confused when it comes to the edge of the road. In the next step, I need to choose a better thresholds method and a better sanity check algorithm to make the program to find the right lane lines.
