---
layout: single
title:  "Texture Fields"
date:   2019-10-18 15:10 +0200
categories: "paper"
tags: ["3D reconstruction", "deep learning", "single view 3D reconstruction", "3D representations"]
author: "Michael Oechsle"
excerpt: >-
    Recently, deep learning methods in the 3D domain have gained popularity in the research community.
    One of the major goals in this area is to reconstruct 3D content from observations.
header:
    teaser: "/assets/posts/2019-10-24-texture-fields/teaser.svg"
---
<p style="text-align: center">
<img src="{{ site.url }}/assets/posts/2019-10-24-texture-fields/header.png" width="35%" />
<img src="{{ site.url }}/assets/posts/2019-10-24-texture-fields/blackroof.gif" width="54%" />
</p>

Recently, deep learning in the 3D domain has gained popularity in the research community.
One of the major goals in this area is to reconstruct 3D content from observations.
For example, we can use neural networks to [estimate the 3D geometry of an object from a single input view]({% post_url 2019-04-24-occupancy-networks %}).
While recent approaches lead to accurate results for estimating geometry, reconstructing the material and texture of objects lacks far behind.
This is surprising as related tasks like synthesizing realistic images are one of the most successful branches of deep learning.

A major reason for the success in the 2D domain is the regular 2D grid structure of images.
This structure allows for using efficient neural networks architectures based on convolutional layers.
In contrast, in the 3D domain, appearance information like texture is defined on the surface of 3D objects.
Unfortunately, we can generally not parameterize the surface with a 2D grid, so that alternative representations are required.

Let us first take a look at two existing representations for texture, colored voxels and texture atlases:

<img src="{{ site.url }}/assets/posts/2019-10-24-texture-fields/reps.png" width="100%" />

### Colored Voxels:
A simple approach for representing geometry and texture is to subdivide space into little colored cells, called voxels.
Similarly to pixels in 2D, the 3D grid structure of voxels enables deep learning architectures based on 3D convolutions.
However, the memory cost of voxel representations grows very fast with the resolution of the 3D grid.
Voxel-based methods are therefore restricted to rather low resolution.
Moreover, especially at low resolution, voxel representations suffer from discretization artifacts and can hence only represent low-frequency texture.

### Texture Atlas:
An alternative approach, which is especially popular in Computer Graphics, is to map every locations on the mesh to a point in a 2D image.
This mapping is called a UV-mapping and the resulting representation of texture is called a texture atlas.
In contrast to colored voxels, a texture atlas is able to represent high frequency details.
However, this representation depends on the UV-mapping that maps a location on the shape to a pixel in the texture atlas.
Unfortunately, there is no unique UV-mapping for arbitrary shapes.
As a result, methods using texture atlases are currently limited to template models with predefined UV-mappings, that are often hand-crafted.


## Challenge
As we have seen, existing texture representations are either limited to low resolutions or do not work well with deep learning.
Can we find a novel representation that
- is independent of the representation of the shape
- does not require expensive space discretization
- does not require UV-mappings
- and can represent high-frequency textures?


## Our Approach
We can understand texture as a mapping that maps any point on the surface of a 3D object to a RGB color value.
Our key insight in this work is that we can directly learn a neural network $t_\theta$ that maps any 3D location to a color value:
$
t_\theta: \mathbb R^3 \to \mathbb R^3
$.
We call $t_\theta$ a Texture Field:

<p style="text-align: center">
<img src="{{ site.url }}/assets/posts/2019-10-24-texture-fields/teaser.svg" width="80%" />
</p>


As it turns out, Texture Fields satisfy all the requirements described in the previous section:
Texture Fields are independent of the shape representation and do not require any discretization during training.
During inference, they can be evaluated at arbitrary resolution, allowing us to represent high-frequency textures.
Our representation also does not require a UV-mapping so that we do not need a fixed template model for shapes.

## Experiments
To evaluate our new representation for texture, we conducted several experiments.

First, we used our novel representation for learning texture reconstruction from a single view by conditioning the Texture Field on an input image and the corresponding 3D model.
We found that our method learns to reconstruct plausible texture and compares favorably to our baselines.
<p style="text-align: center">
<img src="{{ site.url }}/assets/posts/2019-10-24-texture-fields/synthetic.gif" width="100%" />
</p>

Moreover, we combined our method with a [Variational Autoencoder](https://arxiv.org/abs/1312.6114) to obtain a generative model of texture.
Here are some interpolations in the latent space of our generative model:
<p style="text-align: center">
<img src="{{ site.url }}/assets/posts/2019-10-24-texture-fields/interpol.gif" width="100%" />
</p>

## Further Information
If you are interested in Texture Fields, please also check out our video:
{% include video id="pbfeE0qmD2E" provider="youtube" %}

You can download our [paper](http://www.cvlibs.net/publications/Oechsle2019ICCV.pdf) and the [supplementary material](http://www.cvlibs.net/publications/Oechsle2019ICCV_supplementary.pdf).
We also provide [animated slides](https://autonomousvision.github.io/slides/texture-fields/#/) for our project.
If you want to play around with Texture Fields yourself, please take a look at our [source code](https://github.com/autonomousvision/texture_fields).

    @inproceedings{OechsleICCV2019,
        title = {Texture Fields: Learning Texture Representations in Function Space},
        author = {Oechsle, Michael and Mescheder,Lars and Niemeyer, Michael and Strauss, Thilo and Geiger, Andreas},
        booktitle = {Proceedings IEEE International Conf. on Computer Vision (ICCV)},
        year = {2019}
    }
