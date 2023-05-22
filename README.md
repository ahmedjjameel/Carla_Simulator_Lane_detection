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


![fig1](https://github.com/ahmedjjameel/Carla_Simulator_Lane_detection/assets/81799459/9eca275a-f604-47e8-b414-6a0a7c828d0f)


Several transformations are applied to the initial image, which are then intelligently combined together in order to obtain a clear and reliable binary image capable of accurately detecting lane lines. By using the Canny method, it is possible to find all the possible lines in an image, but in addition to the detection of the lane lines the algorithm also provided many edges on landscapes, cars and other objects of no interest which are then discarded. Regarding the search for lane lines, it is known in advance that the lines we are trying to identify can be approximated by vertical lines and both lane lines generally have a high contrast with respect to the road. For this reason, it is possible to take these advantages by using gradients in a smarter and more efficient way to detect steep edges that are more likely to be identified as lanes in the first place.
The Sobel operator is the main core of Canny edge detection algorithm. Applying the Sobel operator to a given image, it is possible to obtain the derivative of the image in the x or y direction in order to filter what we are looking for. Keeping in mind that the kernel size can be represented by any number, as long as it is odd, and using the minimum size of a kernel (which implies a 3x3 operator in each case) the Sobelx and Sobely operators, respectively.

By obtaining the gradients of the images in both the x and y directions, it is possible to notice how these detect lane lines and collect different edges. In particular, the gradient in the x direction highlights the edges closest to the vertical, while the gradient in the y direction emphasizes the edges that are closest to horizontal lines. The Sobel operators combine Gaussian smoothing and differentiation, this implies that the output will be noise resistant.

The Sobel operator used to obtain the gradients both in the direction of x and of y is also used to obtain images with magnitude and direction thresholds. The magnitude of the gradient represents the cornerstone of the detection of the edges of the Canny algorithm and that is why Canny works well to collect all the edges. In particular, it takes the absolute value of the x and y gradients, computed separately, and calculates the direction returning a binary mask where direction thresholds (min and max) are met. The different thresholded binary gradients results images are shown below:


![fig22](https://github.com/ahmedjjameel/Carla_Simulator_Lane_detection/assets/81799459/e3fffdb2-cf7a-4901-a1d8-81d9bf143f5d)

![fig222](https://github.com/ahmedjjameel/Carla_Simulator_Lane_detection/assets/81799459/be02d7de-4320-4316-8e0e-d27a2daf9609)



It is possible to exploit combining the information of the x and y gradient threshold, the amplitude of the general gradient and the direction of the gradient, based on threshold values, to focus on the precise identification of the pixels that determine the lane lines. The combined result is shown below:


![fig333](https://github.com/ahmedjjameel/Carla_Simulator_Lane_detection/assets/81799459/294f2c36-58a5-4b12-a359-e07ca60faf86)


These operations were performed on the original image converted to grayscale to make it easier to detect the edges. However, when performing this conversion, valuable information on color, such as yellow, is lost, which would be useful to give more strength to the identification of lane lines. For this reason, it is useful to take advantage of color spaces that provide more information about an image than just the gray scale in order to consistently detect objects under varying light conditions.
A color space is a way to classify colors and represent them in digital images. This space can be considered as a 3D space, in which any color can be represented by a 3D coordinate of values R (red), G (green) and B (blue). For example, white has the coordinates (255, 255, 255), and therefore has the maximum value for R, G and B. However, RGB thresholding does not work that well in images that include the variation of light conditions or whenever lanes are of a different color like yellow. It works best on white lane pixels. Therefore, it is possible to split an image into separate R­ G-B components, which are often called channels. The brightest pixels indicate higher values of red (R), green (G) or blue (B) respectively. The R and G channels detect white and yellow lane lines and are therefore the most useful for isolating lane pixels, while channel B does not detect the yellow lane line. All channels vary according to different brightness levels.

There are many other ways to represent colors in an image besides the RGB values (Figure 6.5 shows the most common ones) inspired by the human vision system. The most common color spaces used most frequently in image analysis and processing are transformations of a Cartesian RGB color space: the HSV color space (which stands for hue (H), saturation (S) and value (V)) and HLS space (which stands for hue (H), lightness (L) and saturation (S)).
For both of these, H has a range, which span from O to 179 for degrees around the cylindrical color space. HLS Color Space isolates the lightness (L) component, which varies the most under different lighting conditions. H and S channels stay consistent in shadow or excessive brightness.
Hue is the value that represents color independent of any change in brightness, on the other hand, Lightness and Value represent different ways to measure the relative lightness or darkness of a color and finally saturation is a measurement of colorfulness. 


![fig44](https://github.com/ahmedjjameel/Carla_Simulator_Lane_detection/assets/81799459/13ebf4b8-2e16-4ef0-b927-0a62e9224d7c)


Therefore, it is important not only to view each color space channel in such a way as to be able to distinguish and choose the one that is best able to identify lane lines but also use a channel that is more robust than the others and under changing conditions. Assuming that lane lines can be either yellow or white, different color spaces channels were tested using different thresholds.
At the end, among all, 4 different color spaces were used: RGB, HLS, LUV and LAB to help detect lane lines of different colors and under different lighting conditions.
Consequently, it has been concluded that the best combination to detect the lane lines of the road is to use the following channels:
•	The R channel of the RGB color space: it has the appropriate information to identify the white lane lines as well as yellow
•	The S channel from the HLS color space: it was used to highlight the yellow lane lines uniformly
•	The B channel from the LAB color space: it is useful for enhancing the identification of the yellow lanes especially in low light conditions since it has the highest signal­ to-noise ratio
•	The L channel from the LUV color space: it is able to detect white and yellow lane lines almost perfectly
This combination is very effective and robust to detect lane lines in different lighting and road conditions as it offers redundancy and therefore the possibility of detecting the pixels of both lane lines (white or yellow) even in the case where 1 of them should fail. The results are shown below:


![fig55](https://github.com/ahmedjjameel/Carla_Simulator_Lane_detection/assets/81799459/0d4aa2dc-30da-4a2d-bdd5-682be9b6950f)











