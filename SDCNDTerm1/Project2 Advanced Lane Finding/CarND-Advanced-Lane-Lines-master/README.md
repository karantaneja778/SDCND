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

I used previously calculated camera calibration matrix `mtx` and distortion coefficients `dst` to undistort images coming to the pipeline. This is defined as function `undistort(image)` of my `Lane_Detection` class. Here is an example of undistorted image of the road -

![alt text][image1]

#### Step 2 Color and Gradient

From the undistorted image, I converted into HSV color space. By defining upper and lower thresholds on saturation values, it gave me pretty good detection of lane lines. 
I choose to define upper and lower threshold dynamic, based on the average brightness of the image. This helps in detecting lane line in darker and brighter images as well. To find out average brightness, I defined a function `avg_brightness(img)` in Lane_Detection class which returns average brightness ranging from 0 to 255 for a colored image.
To improve more on the edge detections, I applied OpenCV Sobel operator along x-axis and combined the two binary images.
I obtained the following result -

![alt text][image3]

The code can be found in the function `color_gradient(undistorted_image)` of my `Lane_Detection` class.

#### Step 3 Perspective Tranform

The code for my perspective transform includes a function called `warp()` in my `Lane_Detection` class. I chose the hardcode the source and destination points in the following manner:

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
I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto the output of my `color_gradient` image

![alt text][image4]

#### Step 4 Lane Lines Detection

My lane line detection alogrithm consists of three functions `draw_lanes(warped_binary)`, `find_lane_pixels(warped_binary)` and `search_around_lanes(warped_binary)` which are also defined in the `Lane_Detection` class.

Warped image is passed to the `draw_lanes(warped_binary)` function which checks if this is the first time, it will call `find_lane_pixels(warped_binary)` which detect lane lines in the following way -

    * It will divide the warped image into n windows horizontally and start with the bottom window.
    * It will then draw an histogram to detect the peaks in the image. It can be seen as below -

![alt text][image7]

Based on the histogram peaks, it will detect areas of search, based on `margin` defined. in my case, hardcoded it as 100, which worked fine in my case. After search, all the activated pixels are stored and marked as left or right and sent back to `draw_lanes(warped_binary)` function which will calculate 2d polynomial lines.

You can see the highlighted detected pixels as below along with drawn left and right lane lines.
    
![alt text][image5]

Once the left and right lane polynomials are identified, I checked if I have previously stored values, if the number is greater than 3, I do mean of the all the values just to smoothen out the lane lines and make sure odd images in the videos are not causing jitters.

After that all the points between detected left and right lane and identified and highlighted as below -

![alt text][image6]


#### Step 5 Radius of Curvature

Radius of curvature is calculated in my funtion `radius_of_curvature()` and stored as a global variable.

The radius of curvature at any point x of the function x=f(y) is given as follows:

<img src="https://latex.codecogs.com/gif.latex?RCurve&space;=&space;\frac{(1&plus;(2Ay&plus;B)^{2})^{3/2}}{2A}" title="RCurve = \frac{(1+(2Ay+B)^{2})^{3/2}}{2A}" />

I took the max value of y and corresponding x value, and using my left and right lane polynomial equations, I calculated left and right lane curvatures.

Radius of Curvature was then calculated averaging out the left and right curvatures.

#### Step 6 Final results

To get the final results, I have defined `unwarped(binary_img)` which will unwarp the image using the same src and dst vertices but in opposite order. I unwarped the image as follows -

![alt text][image9]

Then this image was masked over the original image and radius of curvature was also put to the resulting image -

![alt text][image8]

---

### Pipeline (video)

#### You can check out the video in which I ran my pipeline. Result looks pretty accurate.

Here's a [link to my video result](./output_video.mp4)

---

### Discussion

#### 
I used all the tools that were taught to me in my Advanced Lane Finding algorithm. I found out the pipeline I created is very slow and it is doing on an average 6-7 iterations per second (3-4 minutes for 50 seconds video) which are not something we can put in the real world.

I am looking forward to learn more tools to make it faster and better


