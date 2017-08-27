# Advanced Lane Line Detection
<a href="https://youtu.be/5QFLcwoe-mM" target="_blank"><img src="http://img.youtube.com/vi/5QFLcwoe-mM/0.jpg" 
alt="IMAGE ALT TEXT HERE" width="240" height="180" border="10" /></a>

## Project Goals

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.


[image1]: /markdown_images/calibration.png "Calibration"
[image2]: /markdown_images/color_spaces.png "Color Spaces"
[image3]: /markdown_images/sobel.png "Sobel Operator"
[image4]: /markdown_images/sobel_gradients.png "Sobel Gradients"
[image5]: /markdown_images/color_targeting.png "Color Targeting"
[image6]: /markdown_images/perspective_transform.png "Perspective Transform"
[image7]: /markdown_images/inital_lanes.png "Initial Lanes"
[image8]: /markdown_images/secondary_lanes.png "Secondary Lanes"
[image9]: /markdown_images/output.png "Output"




## Image Processing Pipeline



### 1. Camera Calibration With OpenCV
### 2. Using New Color Spaces with OpenCV
### 3. Detecting Gradients using the Sobel Operator with OpenCV
### 4. Targeting Specific Colors
### 5. Perspective Transform
### 6. Initally Detecting Lanes Using NumPy
### 7. Restricted Lane Line Search Using NumPy
### 8. Calculating Positional Information
### 9. Draw Lane and Calculated Information


## Camera Calibration With OpenCV



Later on in this project, I will be preforming a perspective transform on input images. For this to work properly I need the input image to be flat. This means that I have to account for and correct the distortion that a camera lens causes.

Calibration works by taking pictures of known shapes, and calculating how they have been distorted. Typically this is achieved by taking a picture of a checkerboard pattern, calculating the coordinates of the corners touching black and white tiles (inside corners), and then calculating the distortion needed to make the points on each row and column fall on a straight line.

Since camera lenses are rigid and don't change, I can take calibration images for a camera and then used the calculated calibration in any setting.

!["Calibration"][image1]


## Using New Color Spaces with OpenCV



Images are typically stored and displayed with the RGB color space, but RGB isn't the best color space for every use case. Later in this project I will be applying a gradient filter to the image to detect lane lines. For best results, the gradient filter needs a lot of contrast between the road and the lane lines. 

RGB works well when the road is black and the lane lines are white, but not so well with shadows, yellow lines, and gray pavement. A better color space for detecting lane lines is HLS, or hue, saturation, lightness. The lightness aspect is just the grayscale version of image, which has an extreme version of the pitfalls of the RGB color space, so I am ignoring that for lane line detection. Hue and saturation could both be used, but saturation is a bit cleaner so I will be using saturation for this project.


![][image2]



## Detecting Gradients using the Sobel Operator with OpenCV



Looking at the saturation image, the lane lines are fairly clear, but there is still noise. For example, the color of the lane is slightly distorted in the middle. The next step is to use gradient thresholding to hone in on the lane lines. To do this, I have to calculate the vertical gradient, and horizontal gradient.

Once I have both gradients I can calculate the angle of the gradient and ignore any gradient that isn't steep, as I expect lane lines to be close to vertical. This will be done by applying the x and y sobel operator to the image. Below is an image of the x and y sobel operator.

![][image3]

These operators are used to estimate the derivative, effectively measuring the steepness of the gradient at every point on the image. This is done by multiplying every 3x3 set of pixels (overlapping) on a one channel image (ie Saturation) and then summing up all the values. From there I can apply a threshold for the intensity of the gradient as well as the angle of the gradient.  

![][image4]

## Targeting Specific Colors



Another method of finding lane lines is to target specific colors. For example, lane lines will often be white and yellow, so I can specifically search for those colors. Although, this isn't as reliable at night, in shadows, or as the lane lines fade, so I can run edge detection on the individual colors as well. This method is then combined with the thresholded saturation.

![][image5]

## Perspective Transform



Later I will be calculating the quadratic formula for each lane line. For this to work I need to warp the image so that two parallel lane lines would be viewed as parallel as opposed to converging on the horizon. This is done by specifying four points on an initial image, and then specifying where those points should end up on the warped image. For this case, I selected four points that make up a trapezoid on the ground that are then mapped to a rectangle. I selected the points based on an image of a lane with straight lines.

![][image6]

## Initially Detecting Lanes Using NumPy



Even if the lane line is curved, the closer the lane is to the car, the more vertical it will be. I can take advantage of that by taking the count of of every x position on the image. This in essence will give me a histogram with one peak on each side of the image. I can then use those peaks as the start of each of the lanes.

From there I take the average of all of the x positions from a window above the previous layer's average. I suggest looking at the image at the bottom of this section before trying to understand the code

Finally, I use NumPy's Polyfit to get a formula for each lane line.

![][image7]

## Restricted Lane Line Search Using NumPy



Once I have a curve that approximates a lane line, on the following frame I can find pixels within a margin of that line, and then recalculate the lane lines.

![][image8]

## Calculating Positional Information 



The instantaneous radius of curvature can be calculated using the formula R = (1+f'(y)^2)^(3/2)/f''(y), where f'(y) = 2Ay + B and f''(y) = 2A given f(y) = Ay^2 + By + C. This formula will give curvature in pixel space so it has to be converted into meters/pixel. Similarly I can calculate the distance from the center of the lane by taking the midpoint of the left and right lanes, subtracting by the center pixel (640) and multiplying by the lane width in meters (3.7)


## Draw Lane and Calculated Information Using OpenCV



The last step is to draw the detected lane line while the image is still warped, unwarp the image, and then mask it over the original image. Finally, I use OpenCV's putText function to write the Positional Information onto the image. 

![][image9]


