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

![self-driving car]({{ site.url }}/assets/posts/2020-03-24-differentiable-volumetric-rendering/wayve.gif){: .align-center}
<sub><sub>Image Source: Wayve.ai</sub></sub>

However, our world is not two- but three-dimensional! If we think about self-driving cars as an example, we can see that autonomous agents need to understand our 3D world to safely interact and navigate in it. They need to __reason in 3D__.

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
which assigns an occupancy probability to every point in 3D space.
The 3D geometry of an object is then implicitly given as the decision boundary of this binary classifier.
We further use a _texture field_
$$
\mathbf{t_\theta}: \mathbb R^3 \to \mathbb R^3
$$
which assigns an RGB color value to every point in space. We implement both $f_\theta$ and $\mathbf{t_\theta}$ in a _single neural network_ with two shallow heads.

### How do we render it?

![forward pass]({{ site.url }}/assets/posts/2020-03-24-differentiable-volumetric-rendering/forward_pass.svg){: .align-center}

If we want to train our network with 2D-based observations like RGB images or depth maps, we somehow need to __render__ our representation.
To this end, we first cast a ray from the camera to pixels and evaluate equally-spaced points on this ray.
Next, We find the interval where points change from outside ($f_\theta(\mathbf{p_j}) < 0.5$) to inside the object ($f_\theta(\mathbf{p_{j+1}}) > 0.5$). Using a root-finding algorithm, e.g. the Secant method, we obtain our final depth prediction.
This gives us a predicted depth map. And how we get the RGB values?

<img src="/assets/posts/2020-03-24-differentiable-volumetric-rendering/forward_pass_rgb.gif" class="align-center" alt="forward pass rgb" width="200px">

Using our predicted depth map, we can now unproject a pixel $\mathbf{u}$ back into 3D space to $\mathbf{\hat p}$.
We simply evaluate our texture field at this point which gives us our predicted RGB color value $\mathbf{t_\theta}(\mathbf{\hat p})$!

### How do we get the gradients?

As we want to train our network end-to-end, we have to provide gradients for all operations with respect to the network parameters $\theta$.
While most calculations are easy to handle, the depth prediction step is tricky as we perform many evaluations. This can be very memory intensive if we need to save all intermediate results. Can we find a more general solution?

We find that we can derive an __analytic expression__ for the gradient of the depth with respect to the network parameters $\theta$.
We first observe that we can express the point on the surface as $\mathbf{\hat p} = \mathbf{r_0} + \hat{d} \mathbf{w}$ where $\mathbf{r_0}$ is the camera origin, $\mathbf{w}$ the ray to the pixel and $\hat{d}$ the predicted depth.
But we also know that our surface is defined as the decision boundary of our occupancy network! Hence $f_\theta(\mathbf{\hat p}) = 0.5$. 
When we differentiate both sides for $\theta$ and plug the first formula in the second one, we get

$$
    \frac{\partial \hat{d}}{\partial \theta} = - \left( \frac{\partial f_\theta(\mathbf{\hat p})}{\partial \mathbf{\hat p}} \right)^{-1} \frac{\partial f_\theta(\mathbf{\hat p})}{\partial \theta}
$$

We have found a way to calculate the gradient for the depth wrt. the network parameters analytically.
This is great as we do not need to approximate the backward pass nor do we need to save intermediate results!

## Does it work?

<p style="text-align: center">
<img src="{{ site.url }}/assets/posts/2020-03-24-differentiable-volumetric-rendering/svr_input.jpg" width="25%" />
<img src="{{ site.url }}/assets/posts/2020-03-24-differentiable-volumetric-rendering/svr_output.gif" width="45%" />
</p>

We find that we can infer accurate 3D geometry and texture from a single image although we only train with 2D or 2.5D supervision!

<p style="text-align: center">
<img src="{{ site.url }}/assets/posts/2020-03-24-differentiable-volumetric-rendering/shape.gif" width="32%" />
<img src="{{ site.url }}/assets/posts/2020-03-24-differentiable-volumetric-rendering/normals.gif" width="32%" />
<img src="{{ site.url }}/assets/posts/2020-03-24-differentiable-volumetric-rendering/texture.gif" width="32%" />
</p>

We further observe that we can use our model for real-world multi-view reconstruction.
We can see that we obtain accurate shape, normal, and texture prediction for this real-world example.

## Further Information
To learn more about Differentiable Volumetric Rendering, check out our video here:
{% include video id="gIha5kvSX9s" provider="youtube" %}

You can find more information (including the [paper](http://www.cvlibs.net/publications/Niemeyer2020CVPR.pdf) and [supplementary](http://www.cvlibs.net/publications/Niemeyer2020CVPR_supplementary.pdf)) on our [project page](https://avg.is.tuebingen.mpg.de/publications/niemeyer2020cvpr). You can further have a look at our [animated slides](http://tiny.cc/3d-deep-learning) and if you are interested in experimenting with DVR yourself, download the [source code](https://github.com/autonomousvision/differentiable_volumetric_rendering) of our project and run the examples. We are happy to receive your feedback!

    @inproceedings{Occupancy Networks,
    title = {Occupancy Networks: Learning 3D Reconstruction in Function Space},
    author = {Mescheder, Lars and Oechsle, Michael and Niemeyer, Michael and Nowozin, Sebastian and Geiger, Andreas},
    booktitle = {Proceedings IEEE Conf. on Computer Vision and Pattern Recognition (CVPR)},
    year = {2019}
    }
