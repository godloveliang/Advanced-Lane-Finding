## Advanced Lane Finding

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

[image1]: ./output_images/undistorted_image.png "Undistorted"
[image2]: ./output_images/original_undistorted_image.png "Road Transformed"
[image3]: ./output_images/combined_binary.png "Binary Example"
[image4]: ./output_images/warp_image.png "Warp Example"
[image5]: ./output_images/fit_polynomial.png "Fit Visual"
[image6]: ./output_images/inverse_wrap_lane.png "Output"
[video1]: ./output_images/project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Provide an example of a distortion-corrected image. Using undistortion matrix obtained at the calibration step, I execute cv2.undistort function to undo the distortion. You can see result below:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I’ve observed that detection works better on S channel of HLS on some set of images, while on others L works better. So I used OR combination to get both. I also noticed that some false positives can be masked by threshold on the S and L channel of HLS, so I applied that step as well.
To strengthen the detect effect, I also used white mask and yellow mask, then used OR combination to the above.
Here is what result looks like:

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp_image()`, which appears in Subtitle 5.  The `warp_image()` function takes as inputs an image (binary_image), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner: the straight lines in the image are parallel，so i drew two horizontal parallel lines and took the four intersections of them and the lane for the source point. 

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:| 
| 435, 560      | 450, 610      | 
| 835, 560      | 850, 610      |
| 1025, 660     | 850, 710      |
| 295, 660      | 860, 710      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for this step is contained in Subtitle 6 "Find lane pixels and fit to find the lane boundary" in advanced_lane_lines.ipynb

If I don’t have prior assumptions about where the lanes are, I do a full search ( find_lane_slide function). First I cut image to 9 pieces horizontally ,and start from the bottom. Then take bottom half of the image and get a histogram and peaks on that histogram. use the left and right peak as the center of the search window, the margin and window_height as the left,right and up search range,find the lane pixels. Mean the x positions to get the next search center,use the same way to deal with the next piece.

If I have information about lane points found in the prior frame, I try to search for lanes near where they used to be (search_around_poly function)

Image below demonstrates polynomial fit for lane points. Left lane is in red, right lane is in blue. Polinomial fit lines are in yellow.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for this step is contained in Subtitle 7 "Determine the curvature of the lane and vehicle position from the center" in advanced_lane_lines.ipynb

First step is to obtain polynomial for the lane lines in meters, not in pixels. For that, knowing the US government requirements for lane width（3.7m） and dashed line distance(3m), I estimated the conversion coefficients and performed a fit.  I use first and second derivative to obtain a curvature.

I take the average of the left and right lane positions as the location of the vehicle. The center of the image is the ideal location. Their difference is the deviation.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

To warp lanes back to original image, I perform a drawing of lanes in the bird eye view image, and then transform it back to a normal view.
The code for this step is contained in Subtitle 8 "Warp the detected lane boundaries back onto the original image" in advanced_lane_lines.ipynb

Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I I tried a lot of color transforms, gradients, etc., to create a thresholded binary image, the result is getting better and better,but the lines are a little wobbly when the car  through some shadowy areas.
If I had more time, I would implement a better binary image processing. I think better results can be achieved by trying more color models and channels. This would give me better results on challenge videos, that can fail due to false positives in the binary image.
