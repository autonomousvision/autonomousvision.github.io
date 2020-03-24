---
layout: single
title:  "Differentiable Volumetric Rendering"
date:   2020-03-24 15:10 +0200
categories: "paper"
tags: ["3D reconstruction", "deep learning", "single view 3D reconstruction", "3D representations"]
author: "Michael Niemeyer"
excerpt: >-
    Deep neural networks have revolutionized computer vision over the last decade. They excel in tasks that are represented in the 2D domain such as object detection, optical flow prediction, or semantic segmentation. However, our world is not two- but three-dimensional! If we think about autonomous robots, self-driving cars, or autonomous drones, we can see that they need to understand our 3D world to safely interact and navigate in it. Autonomous agents need to reason in 3D.
header:
    teaser: "/assets/posts/2020-03-24-differentiable-volumetric-rendering/teaser.png"
---

![teaser image]({{ site.url }}/assets/posts/2020-03-24-differentiable-volumetric-rendering/teaser.png){: .align-center}

Deep neural networks have revolutionized computer vision over the last decade. They excel in 2D-based vision tasks such as object detection, optical flow prediction, or semantic segmentation. 

However, our world is not two- but three-dimensional! If we think about self-driving cars as an example, we can see that autonomous agents need to understand our 3D world to safely interact and navigate in it. They need to __reason in 3D__.

![self-driving car]({{ site.url }}/assets/posts/2020-03-24-differentiable-volumetric-rendering/wayve.gif){: .align-center}
<sub><sub>Image Source: Wayve.ai</sub></sub>

In our previous project [Occupancy Networks](http://www.cvlibs.net/publications/Mescheder2019CVPR.pdf) we asked ourselves: "How should we represent 3D geometry in learning-based systems?" It turns out that using implicit functions to represent 3D geometry is promising and has various advantages over previous representations. For example, we do not discretize space and have infinite resolution. 

## The Challenge
While this works great for synthetic data, we have a problem if want to scale to __real-world scenarios:__ We currently need 3D supervision! This means that during training, we need pairs of images and corresponding 3D objects. This is fine for synthetic objects, but we do not have this data for real-world images!
Therefore, in our recent work [Differentiable Volumetric Rendering](http://www.cvlibs.net/publications/Niemeyer2020CVPR.pdf), we investigate how we can __infer implicit 3D representations without 3D supervision__.

But how do we do it?

## Our Approach
First, let's have a look at how we represent textured objects.
We define two functions, an _occupancy network_
$$
f_\theta: \mathbb R^3 \to [0, 1]
$$
which assigns an occupancy probably to every point in 3D space.
The 3D geometry of an object is then defined as the decision boundary of this binary classifier.
We further use a _texture field_
$$
\mathbf{t_\theta}: \mathbb R^3 \to \mathbb R^3
$$
which assigns an RGB color value to every point in space. We implement both $f_\theta$ and $\mathbf{t_\theta}$ in a _single neural network_ with two shallow heads.

If we want to train our network with 2D-based observations like RGB images or depth maps, we somehow need to __render__ our representation.
To this end, we cast a ray from the camera to pixels and evaluate equally-spaced points on this ray.
We find the interval where points change from outside ($f_\theta(\mathbf{p_j}) < 0.5$) to inside the object ($f_\theta(\mathbf{p_{j+1}}) > 0.5$). Using a root-finding algorithm, e.g. the Secant method, we obtain our depth prediction.

![forward pass]({{ site.url }}/assets/posts/2020-03-24-differentiable-volumetric-rendering/forward_pass.svg){: .align-center}

And how we get the RGB images?

## Does it work?

We conducted extensive experiments on 3D reconstruction from point clouds, single images and voxel grids. We found that Occupancy Networks allow to represent fine details of 3D geometry, often leading to superior results compared to existing approaches.

<p style="text-align: center">
<img src="{{ site.url }}/assets/posts/2019-04-24-occupancy-networks/im2mesh_input.png" width="45%" />
<img src="{{ site.url }}/assets/posts/2019-04-24-occupancy-networks/im2mesh_output.gif" width="45%" />
</p>

## Further Information
To learn more about Occupancy Networks, check out our video here:

{% include video id="w1Qo3bOiPaE" provider="youtube" %}

You can find more information (including the [paper](http://www.cvlibs.net/publications/Mescheder2019CVPR.pdf) and [supplementary](http://www.cvlibs.net/publications/Mescheder2019CVPR_supplementary.pdf)) on our [project page](https://avg.is.tuebingen.mpg.de/publications/occupancy-networks). If you are interested in experimenting with our occupancy networks yourself, download the [source code](https://github.com/autonomousvision/occupancy_networks) of our project and run the examples. We are happy to receive your feedback!

    @inproceedings{Occupancy Networks,
    title = {Occupancy Networks: Learning 3D Reconstruction in Function Space},
    author = {Mescheder, Lars and Oechsle, Michael and Niemeyer, Michael and Nowozin, Sebastian and Geiger, Andreas},
    booktitle = {Proceedings IEEE Conf. on Computer Vision and Pattern Recognition (CVPR)},
    year = {2019}
    }
