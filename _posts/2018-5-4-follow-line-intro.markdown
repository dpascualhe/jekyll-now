---
layout: post
title: Follow Line - Intro
---
And here we go again! This is first post regarding ***Follow Line***, my first assignment for the robotic vision course. The objective of this practice is to implement an algorithm that allows an F1 car to follow a red line painted over a racing circuit and complete a lap. In order to achieve our goal, we're going to use reactive control techniques, more specifically, **PID control**. But first things first. Let's start presenting the tools provided along with this practice.

## Gazebo simulation
Gazebo is an open-source tool for complex robotics simulations. In our case, both the racing circuit and the F1 car, as well as their physical properties, are simulated within a Gazebo world. A screenshot of the *simpleCircuit* world can be seen in the following figure.

![simpleCircuit]({{ site.baseurl }}/images/simpleCircuit.png)

The simulated F1 car is provided with a camera and a laser (those blue rays in the previous image). We're only going to use the first one in this practice.

## *follow_line* 
*follow_line* is the JdeRobot component that links the Gazebo simulation with our algorithm. Besides that, it provides a GUI that shows the live video feed from the car and right next to it a frame to visualize our processing results. This GUI also allows the user to control the car manually.

![follow_line]({{ site.baseurl }}/images/follow_line.png)

And that's it. In the next posts I will present the algorithm developed and the results achieved. See ya!





