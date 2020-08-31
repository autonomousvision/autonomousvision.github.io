---
layout: single
title:  "Convolutional Occupancy Networks"
date:   2020-08-28 14:00 +0200
categories: "paper"
tags: ["implicit neural representationss", "3D reconstruction", "deep learning", "surface reconstruction", "point cloud"]
author: "Songyou Peng"
excerpt: >-
    We introduce a flexible implicit neural representation to perform large-scale 3D reconstruction.
header:
    overlay_image: "/assets/posts/2020-08-28-convolutional-occupancy-networks/con_teaser2.jpg"
    overlay_filter: 0.4
    teaser: "/assets/posts/2020-08-28-convolutional-occupancy-networks/con_teaser.gif"
---

![teaser image]({{ site.url }}/assets/posts/2020-08-28-convolutional-occupancy-networks/con_teaser.gif){: .align-center}

3D reconstruction is a fundamental problem in computer vision with numerous applications, for example, autonomous driving and AR/VR.
With the recent explosive development of deep neural networks, learning-based 3D reconstruction have gained more and more popularity.

In our previous project [Occupancy Networks](http://www.cvlibs.net/publications/Mescheder2019CVPR.pdf) (ONet), we asked the question: "What is a good 3D represention in learning-based systems?" Compared to the explicit representations like point cloud, voxels and meshes, we have shown that implicit neural representations for 3D geometry do not require discretization of the 3D space (e.g. into voxels or points) and are able to represent 3D shapes at arbitrary resolution. See the comparison below, where the rightmost one is result of the implicit representation.

![3d reps]({{ site.url }}/assets/posts/2020-08-28-convolutional-occupancy-networks/3d-reps.png){: .align-center}

## The Challenge
While this works well for single objects, we would also like to scale to __large scenes__. However, the existing approaches condition only on a global feature, which does not capture local information. As you can see below, ONet (left) fails to reconstruct such an indoor scene (right).
<p style="text-align: center">
<img src="{{ site.url }}/assets/posts/2020-08-28-convolutional-occupancy-networks/room_onet_gt.gif" width="100%" />
</p>

Therefore, in our recent ECCV 2020 spotlight paper [Convolutional Occupancy Networks](https://pengsongyou.github.io/conv_onet), we investigate how we can __scale up implicit 3D representations for large-scale scenes__ by incorporating the translation equivariance property of convolutional neural networks.

How do we achieve this goal?

## Our Approach
The idea is surprisingly simple. Given some kind of input, e.g. a sparse and noisy point cloud, we pass it through some feature extractor to acquire point-wise features. Then, we place the feature of every point onto a predefined feature volume according to their 3D coordinates.
<p style="text-align: center">
<img src="{{ site.url }}/assets/posts/2020-08-28-convolutional-occupancy-networks/model_volume.svg" width="100%" />
</p>

The feature volume only contains local features from the sparse input points. In order to acquire also global information, we apply a convolutional hourglass network (U-Net). Now, both local and global information have been integrated into the processed feature volume.

Now, given any point in 3D space, we need to predict if it is occupied. To do so, we can pass the point's 3D point coordinate as well as a feature vector into a fully-connected network to make prediction. Unlike the original ONet that every point uses the same global feature, our method queries a feature for each point from the processed feature volume by trilinear interpolation.
<p style="text-align: center">
<img src="{{ site.url }}/assets/posts/2020-08-28-convolutional-occupancy-networks/model_comp.png" width="80%" />
</p>

This is basically the idea! In addition to the 3D feature volume, we also explore the use of canonical plane(s) to store features. If you are interested, please refer to the paper.

## Does it work?

We first evaluate on a synthetic room dataset, where the input is a noisy point cloud. Compared to ONet (middle), our approach (right) can produce detailed and smooth 3D reconstruction.
<p style="text-align: center">
<img src="{{ site.url }}/assets/posts/2020-08-28-convolutional-occupancy-networks/room_comp.gif" width="100%" />
</p>

We can also take the model trained on this synthetic room dataset, and directly apply to the real-world dataset **ScanNet**.
<p style="text-align: center">
<img src="{{ site.url }}/assets/posts/2020-08-28-convolutional-occupancy-networks/scannet.gif" width="100%" />
</p>

Furthermore, we can also train on small synthetic crops from our synthetic room dataset, and perform 3D reconstruction in a sliding-window manner. In the teaser video, we reconstruct a two-floor building with the size of $15.7m \times 12.3m \times 4.5m$. Below we show another example of the reconstruction of an ancient sites.
<p style="text-align: center">
<img src="{{ site.url }}/assets/posts/2020-08-28-convolutional-occupancy-networks/matterport2.gif" width="100%" />
</p>

## Further Information
To learn more about Convolutional Occupancy Networks, watch our video here:
{% include video id="EmauovgrDSM" provider="youtube" %}

For more detailed information, check out the [paper](https://arxiv.org/abs/2003.04618) and [supplementary](http://www.cvlibs.net/publications/Peng2020ECCV_supplementary.pdf). You can further have a look at our [animated slides](http://tiny.cc/3d-deep-learning) and if you are interested in experimenting with ConvONet yourself, run the [source code](https://github.com/autonomousvision/convolutional_occupancy_networks) of our project. We are happy to receive your feedback!

    @inproceedings{Peng2020ECCV,
        title = {Convolutional Occupancy Networks},
        author = {Peng, Songyou and Niemeyer, Michael and Mescheder, Lars and Pollefeys, Marc and Geiger, Andreas},
        booktitle = {European Conference on Computer Vision (ECCV)},
        year = {2020}
    }