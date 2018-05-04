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
