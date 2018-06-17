---
layout: post
title: 3D Reconstruction - Intro
---

A new exercise from JdeRobot Academy platform! The aim of ***3D Reconstruction*** exercise is to reconstruct a 3D escene given a pair of images captured from calibrated cameras, a.k.a. stereo reconstruction. Broadly speaking, these are steps that we're following to achieve the depth estimation:
1. Features are extracted.
2. Backprojected ray for one feature is computed.
3. Backprojected ray is projected into the image plane of the other optical system to get the epipolar line.
4. Search for correspondences between the neighborhood of the feature evaluated and the neighborhood of the pixels in the epipolar line.
5. Once the correspondence is found, we compute its backprojected ray.
6. The intersection between both backprojected rays is our new 3D point in the reconstructed scene.

...and that's it! The 3D scene that we're going to reconstruct is provided as a Gazebo world, as can be seen in the following picture.

![follow_line]({{ site.baseurl }}/images/3drecon/gazebo.png)

Besides the Gazebo world, a 3D visualization tool for drawing our results is also included in the JdeRobot Academy repo. It's worth mentioning that our cameras form acutally a canonical configuration as they're parallel and pointing in the same direction. This could simplify things a lot but we're not going to make any assumption about the stereo system configuration and try to solve the general task.
