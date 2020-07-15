---
layout: single
title:  "3D Controllable GANs"
date:   2020-07-07 11:10 +0200
categories: "paper"
tags: ["deep learning", "generative adversarial networks", "unsupervised", "3d controllability"]
author: "Yiyi Liao"
excerpt: >-
    We define the new task of 3D controllable image synthesis and propose an approach for solving it by reasoning both in 3D space and in the 2D image domain.
header:
    teaser: "/assets/posts/2020-07-07-controllable-gan/pipeline_comparison_p3.svg"
---

Imagine playing a video game or exploring a virtual reality room, it is essential to observe coherent images when walking around and manipulating objects in the scene.
Currently, image synthesis in these applications is realized using rendering engines (e.g. OpenGL). 

![vr]({{ site.baseurl }}/assets/posts/2020-07-07-controllable-gan/vr.gif){: .align-center}
 <sub><sub>Image Source: https://gph.is/2bknKuG</sub></sub>

While classical rendering tools provide us full control over the 3D shapes, material, lighting etc., 
creating photorealistic content and 3D assets for a movie or video game is very expensive and time-consuming and requires the concerted effort of many 3D artists.

Recently, Generative Adversarial Networks (GANs) have achieved impressive results in photorealistic image synthesis by learning from __raw 2D images__. 
Considering the exciting performance of GANs, we therefore wondered - can we apply GANs to many applications such as virtual reality, simulation and data augmentation?
Unfortunately, vanilla GANs do not automatically expose physically meaningful __3D properties__ such as camera viewpoint, object pose or object entities. This means it is hard to control the generated images of vanilla GANs over these 3D properties.

![gan]({{ site.baseurl }}/assets/posts/2020-07-07-controllable-gan/vanilla_gan.gif){: .align-center}
 <sub><sub>Image Source: https://avg.is.tuebingen.mpg.de/research_projects/convergence-and-stability-of-gan-training</sub></sub>

These observations lead to the core question we investigated in our recent work [Towards Unsupervised Learning of Generative Models for 3D Controllable Image Synthesis](http://www.cvlibs.net/publications/Liao2020CVPR.pdf):
*Can we combine the advantages of classical rendering pipelines and GANs to learn a 3D controllable generative model from raw 2D image observations?*

## 3D Controllable Image Synthesis
First, let's define the task of 3D controllable image synthesis. We are interested in learning a 3D controllable generative model from __2D supervisions only__.
The 3D controllable properties include the 3D pose, shape and appearance of __multiple objects__ as well as the viewpoint of the __camera__.

This is a very challenging task. In this work, we made a first step towards this goal and demonstrated that it is indeed possible to learn a 3D controllable generative model from only 2D images on simple datasets, including both synthetic and real-world scenarios.

![results]({{ site.baseurl }}/assets/posts/2020-07-07-controllable-gan/results.gif){: .align-center}

## Our approach
Inspired by the fact that a 2D image is a projection of the 3D world, our key idea is to model the image generation process jointly in the 3D space and in the 2D images domain. 
Specifically, we first generate some sort of 3D representations via a __3D Generator__. 
Next, we project the 3D representations to the 2D image space via a __differentiable renderer__. 
Finally, these projected 3D representations are converted into a 2D image using a __2D Generator__. 
This is in contrast to vanilla GANs that directly generate images using 2D generators. 

![overview]({{ site.baseurl }}/assets/posts/2020-07-07-controllable-gan/pipeline_comparison_p3.svg){: .align-center}

Once the model is trained, we can control the generated images by manipulating the 3D representations as well as the camera used in the differentiable renderer.
If you are interested in more details, let's now take a closer look at the method.

### How do we control individual objects?
We control individual objects based on a physically meaningful and disentangled 3D representation. 
Specifically, our 3D Generator generates a set of __primitives__ $\mathcal{O}=\\\{\mathbf{o}_1, \dots, \mathbf{o}_N, \mathbf{o}\_{bg} \\\}$.
Here, $\mathbf{o}\_{bg}$ models the scene background and $\mathbf{o}_1, \dots, \mathbf{o}_N$ represent foreground objects. 
For each foreground primitve, $\mathbf{o}\_i=\\\{\mathbf{s}\_i, \mathbf{R}\_i, \mathbf{t}\_i, \mathbf{\phi}\_i\\\}$ with $\mathbf{s}\_i, \mathbf{R}\_i, \mathbf{t}\_i$ determining its pose and $\mathbf{\phi}\_i$ encoding its appearance.
This allows us to manipulate each primitive individually while existing 3D-aware GANs consider the 3D scene as a whole.

<p style="text-align: center">
<img src="{{ site.baseurl }}/assets/posts/2020-07-07-controllable-gan/network_3d.svg" width="50%" />
</p>

### Why primitives and not explicit 3D objects?
You might also wonder why do we learn a set of abstract primitives instead of directly learning the explicit 3D shape and appearance.
In the latter case, the projected 3D primitives can be directly composed as a 2D image that is fed into the discriminator, i.e., without requiring a 2D Generator.
We tried this method by learning deformable 3D spheres. 
We found this method works to some extent, however, it leads to low image fidelity as learning the accurate 3D shape and appearance by deforming spheres is very challenging.

### What type of primitives should we use?
The next question is what type of primitive is suited for this task.
This is an open question and we experimented with different 3D representations, including __point clouds__, __cuboids__ and __spheres__.  
We found these primitives perform very similar in practice.

But what are the advantages of using 3D primitives? Why don't we use 2D primitives? 
Firstly, the 2D primitives provides us only controllability over 2D translations. 
Secondly, the pose parameters of objects in real-world include both translations and rotations. 
We experimentally observe that ignoring the rotation in the model leads to entangled latent representations, i.e., objects might rotate while being translated. 

### How does the 2D Generator work?
We take the projected primitives $\\\{\mathbf{X}\_i, \mathbf{A}\_i, \mathbf{D}\_i\\\}$ as input to a 2D Generator which generates photorealistic images $\mathbf{X}\'\_i$ as well as refined silhouettes $\mathbf{A}\'\_i$ and depth maps $\mathbf{D}\'\_i$. 
Specifically, each projected primitive is fed into the 2D Generator independently with shared weights. 
This allows us to learn a shared 2D Generator across different objects in the same scene. 
We then compose the outputs of the 2D generator using alpha composition according to the depth of the projected primitives.


<p style="text-align: center">
<img src="{{ site.baseurl }}/assets/posts/2020-07-07-controllable-gan/network_2d.svg" width="100%" />
</p>

### How do we train the model?
We train our model with an adversarial loss conditioned on the foreground/background label. 
This enforces the background primitive $\mathbf{o}\_{bg}$ to represent the scene background.
Without the label condition, the model could generate both the foreground and the background using $\mathbf{o}\_{bg}$ alone. 
Note that we need a set of __unpaired__ pure background images for training.

We consider two regularization terms in addition to the adversarial loss.
The first is a compactness loss defined on the projected primitives to enforce the 3D primitives to tightly enclose the objects. 
The second is a geometric consistency loss to regularize the consistency of the objects across different poses. 


## Further Information
To learn more about our work, check out our video here:

{% include video id="NSX-ornLgZI" provider="youtube" %}

You can find more information (including the [paper](http://www.cvlibs.net/publications/Liao2020CVPR.pdf) and [supplementary](http://www.cvlibs.net/publications/Liao2020CVPR_supplementary.pdf)) on our [project page](https://avg.is.tuebingen.mpg.de/publications/liao2020cvpr). Our [source code](https://github.com/autonomousvision/controllable_image_synthesis) is also released. We are happy to receive your feedback!

	@inproceedings{Liao2020CVPR,
	  title = {Towards Unsupervised Learning of Generative Models for 3D Controllable Image Synthesis},
	  author = {Liao, Yiyi and Schwarz, Katja and Mescheder, Lars and Geiger, Andreas},
	  booktitle = { Proceedings IEEE Conf. on Computer Vision and Pattern Recognition (CVPR)},
	  year = {2020}
	}

