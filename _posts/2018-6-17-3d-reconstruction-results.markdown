---
layout: post
title: 3D Reconstruction - Results
---
After implementing the algorithm described in the previous posts, this is the resulting 3D scene reconstruction:

<iframe width="560" height="315" src="https://youtu.be/BqrZqJVbavc" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

The full process for a given pair of frames lasts about 2'30''. It could be probably sped up by applying a more sophisticated optimization technique when trying to find the points that minimize the distance between the backprojected lines, as computing the distance point-by-point is probably overkill. Most of the outliers of the reconstructed 3D scene are discarded with the minimum distance threshold that we imposed. The few outliers that can be seen in the video are most likely due to failures while finding correspondences between the images. Another metric for this process like the correlation of the patches might be more robust than the implemented SSD.
