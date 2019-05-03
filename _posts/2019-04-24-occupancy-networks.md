---
layout: single
title:  "Occupancy Networks - Learning 3D Reconstruction in Function Space"
date:   2019-04-24 15:10 +0200
categories: "paper"
tags: ["3D reconstruction", "deep learning", "single view 3D reconstruction", "3D representations"]
author: "Lars Mescheder"
excerpt: >-
    In recent years, deep learning has led to many breakthroughs in computer vision. Many tasks such as object detection, semantic segmentation, optical flow estimation and more can now be solved with unprecedented accuracy using deep neural networks.
header:
    teaser: "/assets/posts/2019-04-24-occupancy-networks/teaser.png"
---
<div style="text-align: center">
<img src="{{ site.url }}/assets/posts/2019-04-24-occupancy-networks/teaser.png" width="600" />
</div>

In recent years, deep learning has led to many breakthroughs in computer vision. Many tasks such as object detection, semantic segmentation, optical flow estimation and more can now be solved with unprecedented accuracy using deep neural networks.

However, all of these tasks have in common that they can be solved directly in the 2D image domain, allowing us to use powerful 2D-convolutional neural network architectures. But in reality our world is not 2 but 3 dimensional. Reasoning about our world in the 2 dimensional domain is hence only a shortcut which works well for several important tasks in Computer Vision but not all tasks. For example, when we try to apply deep neural networks to 3D reconstruction, we are explicitly interested in a 3 dimensional output and can hence not use the shortcut over a 2 dimensional representation.
But what would be a good output representation for deep neural networks in 3D? In our new paper [Occupancy Networks - Learning 3D Reconstruction in Function Space](https://avg.is.tuebingen.mpg.de/publications/occupancy-networks), we examine this question in depth and propose a new output representation, which allows to apply powerful deep architectures also to the 3D domain.

# The Challenge
In recent years, many groups have investigated different kinds of 3D output representation for learning-based 3D reconstruction. A very natural representation of 3D geometry are voxels. Voxels are a straighforward generalization of pixels to the 3D domain. While using voxels sounds like a very good idea at first, they have  an important drawback which limits their applicability: while the memory requirements for 2D images grows quadratically with the resolution, the memory requirements of voxels grow cubically with the resolution. For example, when we try to convert a fully convolutional architecture for 2D images which operates at resolution 512 x 512 and which fits onto 1 GPU to  a 3D-convolutional architecture operating on voxels, we would need 512 GPUs to satisfy the memory requirements of our network. In practice, this means that architectures operating on voxels are usually restricted to very low resolution such as 32 or 64:

<div style="text-align: center">
<img src="{{ site.url }}/assets/posts/2019-04-24-occupancy-networks/voxels.gif" width="300" />
</div>

Another representation that has been investigated in the past are point clouds.
However, while very flexible and computationally efficient, they lack connectivity information about the output:

<div style="text-align: center">
<img src="{{ site.url }}/assets/posts/2019-04-24-occupancy-networks/pointcloud.gif" width="300" />
</div>

Other works have tried to directly output meshes consisting of vertices and faces. Unfortunately, however, this representation either requires a template mesh from the target domain or sacrifices important properties of the 3D output such as connectivity or does not produce watertight meshes. For example, how would you design a neural network that can output both meshes of chairs and cars? In practice, this is extremely hard, often leading to artifacts such as self-intersections or unnatural outputs:

<div style="text-align: center">
<img src="{{ site.url }}/assets/posts/2019-04-24-occupancy-networks/mesh.gif" width="300" />
</div>

As we have seen, the problem of finding a good 3D output representation for deep learning-based methods is far from solved. We therefore asked ourselves: *can we find an output representation for deep neural networks that*

- can represent meshes of arbitrary topology
- is not restricted to a Manhattan world
- is not limited by excessive memory requirements
- preserves connectivity information
- is not restricted to a specific domain (e.g. object class)
- blends well with deep learning techniques?

Even though this whishlist sound very ambitious, it turns out that it is indeed possible to find a representation of 3D geometry which satisfies all of these requirements.

# Our Approach
In our paper we propose a simple solution to all the requirements mentioned in the previous section. We wondered: When we use deep neural networks for data processing, why not also use them for representing geometry? Instead of artifically separating data processing of the input and the representation of the output, we propose to represent the 3D geometry itself as the decision boundary of a deep learning classifier:


<div style="text-align: center">
<img src="{{ site.url }}/assets/posts/2019-04-24-occupancy-networks/onet.gif" width="300" />
</div>

This simple idea solves all of the problems mentioned in the previous section:
our implicit representation can represent meshes of arbitrary topology and geometry, is not restricted by memory requirements, preserves connectivity information and naturally blends with deep learning techniques.
During training we simply take a random point in 3D and an additional observation (e.g. a single image, a point cloud or a coarse voxelization) as input and classify the point as either occupied or not.
During inference, we propose a simple algorithm, called Multiresolution-IsoSurface-Extraction (MISE) which efficiently extracts meshes from our representation by incrementally building an octree.

# Does it work?

<div style="text-align: center">
<img src="{{ site.url }}/assets/posts/2019-04-24-occupancy-networks/im2mesh_input.png" width="300" />
<img src="{{ site.url }}/assets/posts/2019-04-24-occupancy-networks/im2mesh_output.gif" width="300" />
</div>

Yes, it does. In our paper, we conducted extensive experiments on 3D reconstruction from point cloud, single images and voxel grids. We found that occupancy networks allow to represent fine details of 3D geometry, often leading to superior results compared to existing approaches.

# Further Information
To learn more about occupancy networks, check out our video here:

{% include video id="w1Qo3bOiPaE" provider="youtube" %}

You can find more information (including the paper PDF) on our [project page](https://avg.is.tuebingen.mpg.de/publications/occupancy-networks). If you are interested in playing with occupancy networks yourself, please checkout the [code](https://github.com/LMescheder/Occupancy-Networks) of our project. If you would like to build on our paper or code, don't forget to cite us:

    @inproceedings{Occupancy Networks,
    title = {Occupancy Networks: Learning 3D Reconstruction in Function Space},
    author = {Mescheder, Lars and Oechsle, Michael and Niemeyer, Michael and Nowozin, Sebastian and Geiger, Andreas},
    booktitle = {Proceedings IEEE Conf. on Computer Vision and Pattern Recognition (CVPR)},
    year = {2019}
    }
