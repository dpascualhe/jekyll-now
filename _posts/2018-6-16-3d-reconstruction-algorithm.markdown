---
layout: post
title: 3D Reconstruction - Algorithm
---
We're now going to describe the code implemented to estimate the depth of the given scene. This post is divided according to the steps mentioned in the previous post. Let's go for it.

## Feature extraction
After reading the images from both cameras, we extract features from at least one of them. In our case, we're going to compute the Canny edges filter on the left camera image. The resulting feature map can be seen in the following picture.

![features]({{ site.baseurl }}/images/3drecon/features.png)

Another feature extraction techniques like Harris corner detection, FAST and Good Features to Track have been also evaluated, but the number of extracted points is not enough to reconstruct a recognizable scene.

In order to give the user some control about the density of the map generated, a subsampling step has been added. It simply takes a *subsampling rate* that determines the number of features that survive per feature detected. E.g. if we set the *subsampling rate* a value of 2, then one of each two detected features is taking into account for the reconstruction.

## 3D reconstruction
Once the features, have been extracted, we try to reconstruct the scene one point at a time. We now perform iteratively the next sequence of actions.

### Backprojected ray
Though we probably haven't noticed yet, we have enough information to compute the backprojected ray of each evaluated pixel. A ray is simply a line that has a known start but not an end, and we already know where it starts, i.e. the camera location, provided as an attribute of the *camLeftP* object. We also know the location in the image of the pixel being currently evaluated. If we get it's 3D position, then we have our ray fully defined. In order to get its 3D coordinates, we first transform the 2D coordinates from the digital image reference system, to the camera optical system and then backproject these 2D optical coordinates to the 3D space. 

In order to define this ray, we're going to simply treat it like a straight line and derive its vector equation as follows:

<math xmlns="http://www.w3.org/1998/Math/MathML" display="block">
  <mo stretchy="false">(</mo>
  <mi>x</mi>
  <mo>,</mo>
  <mi>y</mi>
  <mo stretchy="false">)</mo>
  <mo>=</mo>
  <mi>A</mi>
  <mo>+</mo>
  <mi>k</mi>
  <mo>&#x22C5;<!-- ⋅ --></mo>
  <mover>
    <mrow>
      <mi>A</mi>
      <mi>B</mi>
    </mrow>
    <mo>&#x2192;<!-- → --></mo>
  </mover>
</math> 

where *A* will be our camera 3D location, *B* our backprojected point and *AB=B-A*, a vector that defines the direction of our line. As we give different values to the parameter *k*, we generate new points in the line.

### Epipolar line
The epipolar line is simply the projection of the backprojected ray into the image plane of the other optical system, i.e. the defined by the right camera in our case. Our algorithm for computing the epipolar line:

1. Project two 3D points of the backprojected ray into the 2D digital image reference system of the right camera. These points will be the 3D location of the pixel being currently evaluated and a new generated point in its backprojected line. This new point is generated thanks to the vector equation of the backprojected ray with *k=10*.
2. Once we've got two points of the epipolar line, we can define it according to the 2D straight line equation:

<math xmlns="http://www.w3.org/1998/Math/MathML" display="block">
  <mi>y</mi>
  <mo>=</mo>
  <mi>m</mi>
  <mo>&#x22C5;<!-- ⋅ --></mo>
  <mi>x</mi>
  <mo>+</mo>
  <mi>n</mi>
</math>

where the slope *m* will be equal to *A/B* and *n* is the *y* intercept of the line. *A* and *B* will be our two points projected into the digital image reference system.
3. With the equation of the epipolar line properly define, we can now assign values from 0 to the image width *w* to get all the points that conform the epipolar line.

#### Reducing search area
In theory, our candidate point must lie in the epipolar line. However, due to slight deviations in the projection and backprojection processes, it might not be the case. In order to give some error tolerance, a epipolar *strip* is defined. Besides that, we introduce a paremeter that will define the width of the search area from the evaluated pixel. The resultant search region and the original epipolar line can be seen in the following picture.

![epipolar]({{ site.baseurl }}/images/3drecon/epipolar.png)

Notice that the epipolar line is straight because of the canonical configuration of our system.

### Find correspondence
The next step is finding the correspondence between the left pixel being evaluated and its homologous point in the right camera image. In that sense, the sum of squared differences (SSD) is employed as an error metric between the neighborhood of the left pixel and every possible homologous patch in the right image, within the search area previously defined. In our case, the size of these patches is currently set to *7x7*. An example of the result after applying this matching method can be seen in the following picture.

![correspondence]({{ site.baseurl }}/images/3drecon/correspondence.png)

### Correspondence backprojected ray
Following the same procedure than we have mentioned before, the homologous 2D point found in the digital image plane of the right camera is first transformed into the optical plane and then backprojected into the 3D space. As we did before, aided by the vector equation of the straight line the backprojected ray is defined given the right camera position and the backprojected point that we have just computed. Both backprojected rays, for the evaluated pixel and its found homologous, can be seen drawn in 3D in the following picture. 

![backprojected]({{ site.baseurl }}/images/3drecon/backprojected.png)

### Backprojected rays intersection
Ideally, our backprojected rays intersect and that intersection defines our estimated 3D point. But real world is a tough world and that's more than likely not the case. The closer we can get to our solution is to find the points that, while lying in the lines, minimize the distance between them.

In order to find that point, we're going to generate *n=1000* consecutive points of the right image backprojected ray with a parameter *k*  between 0 and 50. Changing *n* we can achieve different resolutions for our estimated point. Once we have all of this points stored, we can now iterate point by point computing its minimum distance to the left image backprojected ray, i.e. a perpendicular from that point to the ray. A nice implementation for computing that distance in Python can be found [here](https://stackoverflow.com/questions/50727961/shortest-distance-between-a-point-and-a-line-in-3-d-space). Then simply the point that gets a minimum distance its our estimated 3D point. Great!

As some errors can arise during the whole process, if our estimated point distance to the backprojected ray is higher than a threshold defined by the user it is just discarded. In our case, we have set that threshold distance to 5.

In the next post I'll be talking about results. See you there!

  