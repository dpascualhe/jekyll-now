---
layout: post
title: Follow Line - Algorithm
---
The algorithm developed relies on computer vision and reactive control to drive the car autonomously. In a nutshell, the red line is segmented from the car front camera in order to check whether the car is centered or not. Then, the error is estimated and fed to a PID control which will tell the car how much to steer to keep on the line. Besides that, the line shape is analyzed to detect if we're on a turn and the velocity is adjusted consequently. The following diagram shows an overview of the method proposed.

![Flow diagram]({{ site.baseurl }}/images/follow_line/flow.png)

## Line segmentation
 The main characteristic that makes the circuit line stand out from the rest of the image is its reddish color, so we're going to perform segmentation as a color filter. In order to avoid issues with changing light conditions, the input image is first transformed to HSV color space, which models luminance and chrominance in different channels. The ranges defined for thresholding the image are the following ones:
 
 ![Color filter]({{ site.baseurl }}/images/follow_line/segmentation.png)

We're not going to take into account the V channel as its too illumination dependent.

Once the line has been segmented, we apply morphological operations to supress holes that might be formed within the segmented line. More precisely, we apply a closing filter with a 7x7 kernel. The result can be seen in the following image.

 ![Segmented line]({{ site.baseurl }}/images/follow_line/segmented_line.png)

## Error computation
For computing the error, a basic approximation of the line displacement with respect to the center of the image is proposed. Given the binary image resultant of the image segmentation process, we analyze white pixels in two rows. The indices of the selected rows are obtained as follows:

 ![Rows indices]({{ site.baseurl }}/images/follow_line/rows.png)

where *h* is the image height. In that way, we make the row selection dependent of the image size. Then, for each selected row, we get the mean location of their white pixels a.k.a. its center. Finally, the following formula is used to compute the error:

 ![Error]({{ site.baseurl }}/images/follow_line/error.png)
 
 In order to make the magnitude of the error easier to work with, we normalize the error dividing it by *w/2*, which is the maximum possible error. In that way, the error will be constrained between -1 and 1.

A visualization of the image center (green line) and the line deviation (blue line) is shown in the following image.

 ![Line deviation]({{ site.baseurl }}/images/follow_line/centers.png)

## Turn detection
In order to detect whether the car is in a turn or not, I tried first to use the same information extracted for computing the error. However, it resulted to be tricky as sometimes, when you're in a turn, the upper and lower centers defined before can be well aligned with the image center. Surprisingly, the most effective method I have come up with is analyze the segmented line geometry. Thanks to OpenCV *findContours* and *approxPolyDP* functions, it is possible to find the contour of the line object and approximate a polygon with a given precision. As it can be seen in the following pictures, when the line is straight its shape resembles a triangle, but when we're on a turn its shape turns into a more complex polygon.

 ![Different lines]({{ site.baseurl }}/images/follow_line/straight_turn.png)

Taking advantage of this property, we detect a turn when the approximated polygon of the line has more than 3 sides.
 

