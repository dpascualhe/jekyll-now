---
layout: post
title: Follow Line - Algorithm
---
The algorithm developed relies on computer vision and reactive control to drive the car autonomously. In a nutshell, the red line is segmented from the car front camera in order to check whether the car is centered or not. Then, the error is estimated and fed to a PD controller which will tell the car how much to steer to keep on the line. Besides that, the line shape is analyzed to detect if we're on a turn and the velocity is adjusted consequently. The following diagram shows an overview of the method proposed.

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
 
## Reactive control
When we talk about reactive control, we mean that our robot behavior will be constantly determined by its sensors output. This kind of behavior can be summed up in a closed loop like the following one:

 ![Reactive control]({{ site.baseurl }}/images/follow_line/reactive_control.png)
 
 This kind of control is suitable to our problem as we need a really fast interaction with the environment. Besides that, our only sensor is a camera and we only have to fulfill one task: follow the red line. This low-complexity system can be easily controlled in a reactive manner. For this practice, we're going to implement a PD controller which is a control loop as the one we have seen before, but aided by feedback about whether or not the error is decreasing. It relies in two terms: proportional and derivative. On one hand, the proportional term just takes the error and scales it by a given factor or gain. On the other hand, the derivative term scales not the error, but its derivative with respect to the time, which can be easily obtained as the difference between the current error and the error in the previous frame. Including a derivative term is necessary because otherwise the controller output would oscillate instead of converging to the desired one. The following equation summarizes the PD control we have just explained:
 
  ![PD controller]({{ site.baseurl }}/images/follow_line/pd.png)

Every order we send to the car actuators will be based on the PD controller output and the *turn dectector* that we have implemented. In order to achieve our objective, four different cases with their respective actuations have been defined:

<style>
.tablelines table, .tablelines td, .tablelines th {
        border: 1px solid black;
        }
</style>

| Situations                       | Actions                                    |
| -------------------------------- | ------------------------------------------ |
| Turn                             | v = 6;    w = PD controller(kp=2, kd=3.5)  |
| Straight line during > 10 frames | v = 20;   w = PD controller(kp=0.25, kd=3) |
| Straight line during < 10 frames | v = 6;    w = PD controller(kp=0.25, kd=3) |
| No line                          | v = 0.25; w = 0.25                         | 
{: .tablelines}

All the values for the proportional and derivative terms gains have been adjusted experimentally, as well as the linear velocities for each case. Now I'm going to clarify some decisions that may look weird:
- When the car is over a straight line, I have decided to take into account how much time has passed since we found the last turn as I noticed that, sometimes, when two turns are consecutive and heading to opposite directions, there's a small section of line between them that looks straight. That little straight area would made the car accelerate way too much, losing the red line.
- When we cannot see the line, we just start spinning slowly until we found it again. It doesn't work quite fine, but it's the best solution I've come up with :(.

In the following post we're going to see the F1 car hit the pedal to the metal. See you there!