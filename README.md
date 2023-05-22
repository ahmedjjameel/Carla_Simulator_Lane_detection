## Carla Simulator Lane Detection
The fundamental requirement for the implementation of lane detection systems lies in calculating the exact position of the vehicle with respect to the lane lines. For this purpose, it is necessary to use in-depth computer vision techniques to recognize particular features used to determine lane detection.
This part is entirely dedicated to the analysis of the lane lines identification, which allows adding useful information such as the radius of curvature of the lane and the position of the vehicle with respect to the lane center, using OpenCV library. 
The steps that exploit various and complex artificial vision techniques to extract particular features of road lanes from image frames are the following:

1.	Read the frame image and resize it to the appropriate resolution
2.	Apply the combined Gradient Thresholds to the image frame returning a binary output image to detect edges
3.	Apply the combined Color Thresholds to the image frame in order to create a binary image that correctly identifies the pixels that represent lane lines through color selection
4.	Merge both solutions of points 2 and 3 in a single binary image
5.	Apply a perspective transform (BEV) using the ROI vertices to correct the binary image
6.	Apply a lane lines finding method using histogram peaks and sliding window
7.	Identify lane line pixels fitting their positions using a second order polynomial fit to find the lane boundary
8.	Determine the radius of curvature of the lane and the offset of the vehicle with respect to the center of the lane
9.	Use the inverse perspective transformation to get back the original image and draw the lane lines on it
10.	Display the final result with the information associated to the point 8

In order to be able to extract particular information from a lane in an image in a reliable and precise way, it is necessary to look for and highlight some typical features in the image. First, the image is read and resized to HD resolution as shown below through OpenCV library in such a way to be able to subsequently identify more easily the ROI vertices.
