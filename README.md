## Carla Simulator Lane Detection

![gif1](https://github.com/ahmedjjameel/Carla_Simulator_Lane_detection/assets/81799459/3db5fd20-a440-4c2e-8c01-13fe4fbcf748)  |   ![gif2](https://github.com/ahmedjjameel/Carla_Simulator_Lane_detection/assets/81799459/bcf975b5-066d-4d58-8630-07f5adea52b8)
:-------------------------:|:-------------------------:

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

These channels are filtered through a logical AND operator ("&") with particular thresholds that are different for each type to correctly isolate the pixels of the lane lines. Threshold values strongly depend both on the type of image being analyzed and on the type of channel taken into consideration. After that, their result is combined together via a logical OR operator ("|") to make the binary output image robust and reliable in detects lane lines pixels even with different colors.

![fig66](https://github.com/ahmedjjameel/Carla_Simulator_Lane_detection/assets/81799459/aa16fbc8-9298-476e-aff5-09af2da5a34b)


Finally, to minimize false detections and make identification even more effective, an OR operation was performed between color binary image and gradient binary image previously calculated (combining them), putting together in a single image the peculiarities of both solutions as shown below:

![fig77](https://github.com/ahmedjjameel/Carla_Simulator_Lane_detection/assets/81799459/b779553c-08e8-4612-80b9-17a9ac7a2b4b)

Using the current perspective space is not convenient for the calculation of lane distances. For this reason, to eliminate the perspective effect and simplify the calculations it is useful to exploit the perspective transformation, in particular the bird's eye view, which represents a valid tool both for the detection and for the adaptation of the lanes.
Assuming that the lane lines lie on a flat 2D surface, it is possible to take advantage of a bird's eye perspective transformation (that is, observe the same image from above) both to see the parallel lane lines (more or less depending on curvature), and to enlarge those lane lines distant from the vehicle, which otherwise would be small and therefore difficult to identify. This is extremely useful, especially for road images, since it allows visualizing the lanes from above in such a way that it is easier not only to apply model fitting to this top-view image that can accurately represent the lane, but also to calculate the curvature of the lane and the vehicle's offset.

The idea behind the perspective transformation is based on mapping the pixels of the original image (front view) to an image with a new perspective (viewed from above). The BEV (bird's eye view) perspective transformation can be applied to an image through 2 openCV function: getPerspectiveTransform() to calculate a perspective transform matrix based on four pairs of points (source points and destination points) and warpPerspective() function to apply it on the original image. These 2 functions are used by the binary_transform() function, which through the source and destination points computes the perspective transformation matrix and its inverse, and applies it to the binary image previously obtained, so returning a binary warped image.

The source points of interest represent the 4 coordinate points in the original image that lie on the plane in the physical world that are mapped to the destination points of interest, which represent the 4 coordinate points in the warped image. It is necessary to note that the four source points must be chosen accurately as they represent the fundamental task of selecting the region of interest (ROI) having the typically trapezoidal shape as shown below:


![fig88](https://github.com/ahmedjjameel/Carla_Simulator_Lane_detection/assets/81799459/be7654b1-3479-4aee-a09b-72e50d574285)

In particular, the upper part of the trapezoid must be sufficiently high in the original image to be able to detect the lanes located near the horizon to allow the algorithm to both adequately calculate the curvature of the lane and to increase the number of line segments in order to perform a better fit. To search for these points correctly and quickly, the find_ROI_coordinates.py python script can be used.


![fig99](https://github.com/ahmedjjameel/Carla_Simulator_Lane_detection/assets/81799459/f705cbb8-d519-433c-8fc2-adb78c52c828)



It is important to note that the trapezoid deriving from the coordinates of the source points in the original image is transformed (mapped) into a rectangle in the warped image.
Another thing that is worth noting is that, when the algorithm reconstructs the warped image through the bird's eye view, this is more blurred in all those points where the lane lines are more distant than the vehicle because there are fewer pixels for the reconstruction bringing the algorithm to lower the resolution. By obtaining the perspective transformation, it is possible to visually check whether the lines are approximately straight or not. After applying a correct perspective transformation to the road image, the output obtained will be a binary image in which the lane lines stand out visibly as shown below: 


![fig10](https://github.com/ahmedjjameel/Carla_Simulator_Lane_detection/assets/81799459/bcbc98a3-a8d7-43c3-b51c-1c73e3272d4d)


However, it is necessary to establish which pixels must be part of the lane lines and especially which ones belong to the left or right line. A quick and efficient solution to this problem lies in representing a histogram capable of tracing binary activations in order to find the starting point of lane lines, using get_histogram() function,  fulfills this work. In this way, using the histogram, the values of each pixel are summed along each column in the image, since lane lines are likely to be mostly vertical nearest to the car.
In the deformed binary input image, the pixels forming the lane lines are represented by white (logic value 1) instead the pixels that are not part of them are represented by black (logic value 0). Thus, the two most important peaks (i.e. those with the highest value) in the histogram can be easily identified as candidates that will indicate the x position of the starting point of the lane lines. The result is shown below:

![fig11](https://github.com/ahmedjjameel/Carla_Simulator_Lane_detection/assets/81799459/7d54e520-d58e-4055-bdb6-b5667596122b)

The histogram is divided exactly in half, based on its midpoint value, a left and a right part in order to do the computation more easily. Then the maximum point in each part is calculated, which will be taken into consideration when the windows around these maxima will be created. In this way, it is possible to exploit together the deformed binary image and the histogram to be able to use the sliding window method. It is placed in the central position of the lane lines for both the right and left lanes, and it is used to find and follow the lines to the top of the entire image frame (i.e. at the maximum height) including the lines distant from the vehicle further along the road to determine their direction.
The steps to execute the sliding windows method described below are searchable in the sliding_window(), skip_sliding_window() and display_poly() functions for Python program.
Firstly, the histogram of the bottom half of the image is taken and it is split into two sides, one for each lane line in order to get the left and right peak. These peaks will represent the starting point to reconstruct the left and the right lines. After entering the parameters a priori to set the appearance of the windows, also called hyperparameters (like the number of sliding windows, the height of windows, the width of the windows margin, the minimum number of pixels found in order to re-center window), and having previously obtained the lane lines starting points using the histogram, it is possible to trace the curvature of the right and left lane lines.

For both the right and left lane lines, the window will scroll to the left or right if it detects that the average position of the pixels activated within the window (i.e. if there are white pixels identifying the line) has moved. The steps required to implement the sliding window method are the following:

1.	Loop through each window in nwindows (total number of sliding windows)
2.	Identify the current window boundaries, this is based on a combination of the current window's starting point ( leftx_current and rightx_current), as well as the margin setted before
3.	Draw windows on the image using cv2.rectangle() OpenCV function to be able to visually locate them in the space
4.	Find out which activated pixels (nonzero pixels in x and y) from nonzeroy and nonzerox are inside the window
5.	Enter these values in a list of indexes (to left_lane_inds and right_lane_inds), both for the right and left lane line
6.	If the number of pixels found in step 4 is greater than the minpix parameter (i.e. the minimum number of pixels found needed to re-center window), re-center the window (ie leftx_current or rightx_current) based on the average position of these pixels
7.	Extract left and right line pixel positions storing their x and y coordinates into a vector (leftx, lefty, rightx, righty)

After finding all the pixels belonging to each lane line through the sliding window method, it is necessary to fit a polynomial to both line. In particular, polyfit(), a OpenCV function, applies a least squares polynomial fit of order 2 to the lane lines, returning polynomial coefficients. To identify the pixels of the lane lines, their x and y pixel positions were used to adapt to a polynomial curve of the second order in the polyfit equation form:

$𝑓(𝑦)=𝐴𝑦^{2}+𝐵𝑦+𝐶$

![fig12](https://github.com/ahmedjjameel/Carla_Simulator_Lane_detection/assets/81799459/696c4df4-9ca9-4deb-9915-168d2d4eec8f)

It has been adapted f (y), rather than f (x), because the lane lines in the image are deformed almost vertical and may have the same x value for more than a y value. Once the method that uses the sliding windows to trace the lanes in the distance has been created, it is necessary to keep in mind that the use of this for each frame (for example of a video) could take a long time for the processing and therefore be inefficient.

Since the lane lines do not move much from one frame to another in a video, it is not necessary to search for lanes again using the algorithm from the beginning, but more intelligently it is possible to search in a margin (set a priori) around to the position of the previous line (skipping the sliding window). This works as a sort of "Kalman filter" to predict the position of the lane lines in the next frame within an area of margin defined a priori. In this way, knowing exactly the position of the lines in a frame, it is possible to perform a targeted search around them in the next frame. This makes it easier to follow lane lines, as this method is equivalent to applying a custom region of interest for each frame. In particular, polynomial functions are used to determine which activated pixels fall within the areas of interest established by a margin (+/-).

![fig13](https://github.com/ahmedjjameel/Carla_Simulator_Lane_detection/assets/81799459/472563ed-1603-40d7-861f-8cf690ed3836)

Given the polynomial fit for the left and right lane lines, the radius of curvature for each line is calculated according to formulas presented [here](https://www.intmath.com/applications-differentiation/8-radius-curvature.php). Also, the distance units from pixels to meters is converted, assuming 30 meters per 720 pixels in the vertical direction, and 3.7 meters per 700 pixels in the horizontal direction.
Finally, the radius of curvature for the left and right lane lines is averaged, and reported this value in the final video's annotation.


![fig144](https://github.com/ahmedjjameel/Carla_Simulator_Lane_detection/assets/81799459/ee9c3cde-171b-4ed7-ac88-d94ef20a4d21)


The radius of curvature was initially calculated based on the values of the pixels, then it was decided to choose a more realistic measurement that reflects reality and therefore before executing the calculation of the radius of curvature, the x and y coordinates have been converted from pixel space to real world space. This involves measuring the length (y) and width (x) of the lane section we are projecting into our deformed image. This data was extracted from CARLA simulator. In this way, using the measure_curvature() function, the radius of curvature of the right and left lane lines in meter has been calculated by fitting a new polynomials to x and y in world space, computing also the difference and the average between the two curvatures.
Moreover, it is also possible to calculate the distance and direction of the vehicle with respect to the center of the lane, in order to know exactly where it is. This can be done, based on two assumptions:

 
•	The camera is mounted in the center-front part of the car, so that the center of the lane corresponds to the midpoint in the lower part of the image between the two lines detected
•	The width of the road lane must be known (3.5 meters in Carla Simulator)

The offset of the center of the lane from the center of the image (converted from pixel to meters) represents the distance of the vehicle from the center of the lane as shown below:

![fig15](https://github.com/ahmedjjameel/Carla_Simulator_Lane_detection/assets/81799459/e4b57e5e-546e-4e8c-9d60-8e3ae176c413)

The offset of the car from the center of the lane was obtained by averaging the coordinates of the starting points for identifying the left and right lane lines, subtracting the latter from the middle point of the camera. Finally, to obtain a compliant measurement, the result is multiplied by the so called pixels-to-meters ratio, i.e. the ratio between the effective width in meters of the lane (3.5m in this case) and the pixels of the lane (720) so as to obtain a conversion from pixels space to meters (world space).

In addition, based on the offset value, it is also possible to understand in which direction the vehicle is moving in relation to the centre of the lane: if the offset is positive, the vehicle is positioned to the right with respect to the centre of the lane, and vice versa. 
The final result was thus obtained that is shown below:

![fig1-1](https://github.com/ahmedjjameel/Carla_Simulator_Lane_detection/assets/81799459/4bb64a4a-7534-4465-b8fc-9e307ed73b48)  |  ![figg1-1](https://github.com/ahmedjjameel/Carla_Simulator_Lane_detection/assets/81799459/d6544227-9a79-4cfc-aeaf-06e3a1e18088)
:-------------------------:|:-------------------------:

### Output Video
After applying various algorithms, we will obtain results after plotting everything on the Image and writing it into video.

![gif1](https://github.com/ahmedjjameel/Carla_Simulator_Lane_detection/assets/81799459/3db5fd20-a440-4c2e-8c01-13fe4fbcf748)  |   ![gif2](https://github.com/ahmedjjameel/Carla_Simulator_Lane_detection/assets/81799459/bcf975b5-066d-4d58-8630-07f5adea52b8)
:-------------------------:|:-------------------------:
