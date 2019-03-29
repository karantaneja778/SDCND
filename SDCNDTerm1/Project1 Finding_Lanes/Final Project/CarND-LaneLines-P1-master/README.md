# **Finding Lane Lines on the Road** 
---
[//]: # (Image References)

[image1]: ./test_images_output/gray.jpg "Grayscale"
[image2]: ./test_images_output/blur_gray.jpg "Gaussian Blur"
[image3]: ./test_images_output/edges.jpg "Canny Edge Detection"
[image4]: ./test_images_output/masked_edges.jpg "Region of interest"
[image5]: ./test_images_output/lines.jpg "Hough Transform"
[image6]: ./test_images_output/masked_image.jpg "Final Image"
[image7]: ./test_images/solidYellowCurve2.jpg "Original Image"
**Finding Lane Lines on the Road**

### Original Sample Image
![alt text][image7]
---

### Reflection

### 1. Pipeline to detect lane lines and draw a mask over it.

My pipeline consisted of six steps. 

First, I converted the images to grayscale

![alt text][image1]

Second, I applied Gaussian Blur to smoothen the image

![alt text][image2]

Third, Edges were detected in the smoothen image using Canny Edge Detection.
In my case, lower and upper threshold values 50 and 150 worked fine.

![alt text][image3]

Fourth, I defined region of interest for which I put a triangular mask on the edges image, this gave me approximation of the lane that needs to be detected.

![alt text][image4]

Fifth, I applied Hough transform to draw straigh lines over left and right lane

In order to draw a single line on the left and right lanes, I modified the draw_lines() function as follows -
1. Identified all x and y for left and right lanes based on the slopes of the lines detected by hough transform. Negative slopes gave me left lane and similary positive slope gave me right lane.
2. After doing various tests, I came to assumption that for detecting lanes in straight roads, I can consider only slopes between 0.5 and 1 for right lanes and between -1 and -0.5 for left lane. This gives me more accurate results for the project videos.
3. After getting all x and y, I used numpy polyfit and poly1d functions to get 1d equation of straight lines for left and right lane.
4. I took starting point of the left and right lanes as max y, i.e. starting from the bottom of the image.
5. Based on above y, and using 1d equation corresponding x values are calculated for left and right lanes
6. Similarly for end point of the lane, I used center of the x-axis and calculated corresponding y values for left and right lanes.
7. For left lane, remove 20 pixels form x before calculating y value to remove the intersection of left and right lanes. Similary for right lane add 20 pixels.
8. Last step was to just draw straight lines using opencv line function to display left and right lanes.

![alt text][image5]

Last step was to add these lines to the original image and final results were obtained.

![alt text][image6]



### 2. Potential shortcomings with the current pipeline

My pipeline is based on the assumption that the input images/videos consist of car going straight on the road when camera mounted in the center front of the car.
This will not work on turns and bends.

### 3. Possible improvements to the pipeline

This can be improved by the following steps-

1. Vertically divide image into two parts, left and right.
2. Apply step 1, 2 and 3 of my current pipeline
3. For left and right images individually, detect points of interest.
       i. For left lane, get all the lines using hough transform and then get x and y for left lanes from the lines which are nearest to the line x = x_max
       ii. For right lane, get all the lines using hough transform and then get x and y for right lanes from the lines which are nearest to the line x = 0
4. Then create lane lines from the identified points and mask on original image.

