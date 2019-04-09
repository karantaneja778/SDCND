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

[image1]: ./output_images/undst.jpg "Undistorted"
[image2]: ./output_images/original.jpg "Input Image"
[image3]: ./output_images/clg.jpg "Binary Example"
[image4]: ./output_images/warped.jpg "Warp Example"
[image5]: ./output_images/lanepixels.jpg "Left and Right Lane Detection"
[image6]: ./output_images/lane.jpg "Lane Fill"
[image7]: ./output_images/histogram.jpg "Histogram"
[image8]: ./output_images/result.jpg "Output Image"
[image9]: ./output_images/unwarped.jpg "Unwarped Binary"
[image10]: ./examples/undistort_output.png "Undistort Chessboard"
[video1]: ./output_video.mp4 "Video"

### README

My `Lane Detection` algorithm is written as a python class here "./P2.ipynb" which can be used by instantiated an object of this class and calling `process_image_pipeline()` function.
```python
lane_detection = Lane_Detection()
output_image = lane_detection('test_images/test3.jpg')
```
It also gives user capability to define buffer size. This buffer size number will define how many images will be taken and average of the detected lanes will be taken and mask over original image. This helps remove jitters and smoothen out lane detection. By default it takes value of 5 during object creation of my class. This can be changed during object creation as follows:
```python
lane_detection = Lane_Detection(buffer_size=10) # this will take 10 images and average it out.
output_image = lane_detection('test_images/test3.jpg')
```
### Camera Calibration

The code for this step is contained in the first function definition of the class Lane_Detection which is located in file "./P2.ipynb". The function name is `camera_calibration()`. This function is called every time an object of the class is created and stores "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. 

Then I used the output `objpoints` and `imgpoints` to compute the camera calibration matrix `mtx` and distortion coefficients `dst` using the `cv2.calibrateCamera()` function and store these values 
To verify the result I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained the following result: 

![alt text][image10]

### Pipeline (single image)

## Original image

![alt text][image2]

#### Step 1 - Distortion correction

I used previously calculated camera calibration matrix `mtx` and distortion coefficients `dst` to undistort images coming to the pipeline. This is defined as function of my `Lane_Detection` class `undistort(image)`. Here is an example of undistorted image of the road -

![alt text][image1]

#### Step 2 Color and Gradient


![alt text][image3]

#### Step 3 Perspective Tranform

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
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
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### Step 4 Lane Lines Detection

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image7]

![alt text][image5]

![alt text][image6]


#### Step 5 Radius of Curvature

I did this in lines # through # in my code in `my_other_file.py`

#### Step 6 Final results

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image9]

![alt text][image8]

---

### Pipeline (video)

#### You can check out the video in which I ran my pipeline. Result looks pretty accurate.

Here's a [link to my video result](./output_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
