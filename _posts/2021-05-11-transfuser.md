---
layout: single
title:  "Sensor Fusion for Self Driving"
date:   2021-05-11 20:45 +0200
categories: "paper"
tags: ["self driving", "autonomous vehicles", "imitation learning", "sensor fusion", "transformers"]
author: "Aditya Prakash"
excerpt: >-
    We use attention-based feature fusion to combine image and LiDAR representations.
header:
    teaser: "/assets/posts/2021-05-11-transfuser/teaser.gif"
---

![scenario]({{ site.url }}/assets/posts/2021-05-11-transfuser/scenario.svg){: .align-center}

Consider a scenario where the ego-vehicle (shown in green) is about to enter an intersection. To safely navigate the intersection, the ego-vehicle needs to understand the relation between the traffic lights on the right (shown in yellow) and vehicles on the left (shown in red), e.g. if the traffic light state is green then the vehicles on the left will move towards the right side of the intersection. If the ego-vehicle is unable to capture this relationship, it could lead to dangerous collisions. Therefore, in a dense urban setting with multiple vehicles and traffic lights, it is imperative for any vehicle to model the dependencies between different entities in the entire 3D scene, i.e., the ego-vehicle needs to capture the *global context of the 3D scene*.

![sensors]({{ site.url }}/assets/posts/2021-05-11-transfuser/sensors_overview.svg){: .align-center}

The ego-vehicle perceives the environment through different sensors - cameras and LiDAR being the most popular ones. While a camera provides dense perceptual information of the scene, it lacks reliable 3D information and is highly susceptible to variation in weather conditions. On the other hand, LiDAR consists of 3D information but LiDAR measurements are typically very sparse (in particular at distance) and do not contain important information such as traffic light state. Hence, image-only and LiDAR-only methods are likely to fail in complex urban scenarios.

This limitation can be mitigated by *integrating information from camera and LiDAR sensors* since their benefits are complementary to each other. This leads to several research questions which we investigate in this work:
1. How to integrate information from multiple sensors?
2. To what extent should they be processed independently?
3. What kind of fusion mechanism to use for maximum gain?

## Geometric Fusion

![gf_visual]({{ site.url }}/assets/posts/2021-05-11-transfuser/gf_visual.svg){: .align-center}

Prior works on *sensor fusion* are mostly focused on geometry-based fusion between camera and LiDAR sensors. In this method, points in 3D space (LiDAR point cloud) are projected to pixels in image space (camera input) and information is aggregated from the projected locations. In particular, the features (extracted using a convolutional neural network) corresponding to these projected locations are combined together. This is referred to as *geometric feature projection* in the above figure. This has been shown to be very effective on vision tasks such as [object detection](https://arxiv.org/pdf/2008.11901.pdf), [motion forecasting](https://arxiv.org/pdf/2008.11901.pdf) and [depth estimation](https://arxiv.org/pdf/2009.01875.pdf) but it has not been extensively explored in the context of end-to-end driving.

## Limitation of Geometric Fusion

We observe that geometric fusion underperforms in complex urban scenarios involving dense traffic resulting in collision with other agents in the scene.

![gf_limitation]({{ site.url }}/assets/posts/2021-05-11-transfuser/gf_limitation.gif){: .align-center}

We hypothesize that this happens due to the lack of global context since features are aggregated from a local region in the projected 2D or 3D space. In the illustraion shown below, for the traffic light region in the image (shown in yellow), geometric fusion aggregates features from the blue region in LiDAR point cloud since these points project to the yellow region in image space. However, to safely navigate the intersection, it is essential to aggregate features from the red region in LiDAR point cloud since it overlaps with vehicles moving from left to the right.

![gf_limitation_visual]({{ site.url }}/assets/posts/2021-05-11-transfuser/gf_limitation_visual.svg){: .align-center}

## Our Approach

Our key idea is to use *attention-based feature fusion* which helps the network to aggregate features from relevant regions in both the sensors. These relevant regions are spread across the entire input space rather than being restricted to just the projected locations (as in geometric fusion). This helps to capture the global context of the entire 3D scene. In particular, we use [*Transformers*](https://arxiv.org/pdf/1706.03762.pdf) for this purpose - [*Multi-Modal Fusion Transformer (TransFuser)*](https://arxiv.org/pdf/2104.09224.pdf).

![transfuser]({{ site.url }}/assets/posts/2021-05-11-transfuser/transfuser.svg){: .align-center}

We consider RGB image and LiDAR BEV (topdown view of the LiDAR point cloud) as inputs to our model. These are then processed by convolutional neural networks (in particular, [*ResNet*](https://arxiv.org/pdf/1512.03385.pdf) modules) resulting in intermediate feature maps of different resolutions. We then use transformers to combine the image and LiDAR features at multiple resolutions. TransFuser outputs a feature vector which is then passed to a GRU-based autoregressive waypoint prediction network. The predicted waypoints are then fed to PID controllers which output the vehicle control.

## Results

We conduct extensive experiments using [CARLA](http://carla.org/) in dense urban setting involving [complex scenarios](https://leaderboard.carla.org/scenarios/) such as pedestrians emerging from occluded region to cross the road at random locations, other vehicles running red light at intersections and unprotected turnings.

#### Driving Performance in New Town

![gen_nt]({{ site.url }}/assets/posts/2021-05-11-transfuser/gen_nt.gif){: .align-center}

#### Driving Performance in New Weathers

![gen_nw]({{ site.url }}/assets/posts/2021-05-11-transfuser/gen_nw.gif){: .align-center}

#### Infraction Analysis

<center><img src="{{ site.url }}/assets/posts/2021-05-11-transfuser/infraction_analysis.svg" width="500"></center>

Our experiments indicate that TransFuser incurs fewer collisions and red light violations and can navigate difficult scenarios.

## Further Information
To learn more about work, watch our video:

{% include video id="WxadQyQ2gMs" provider="youtube" %}

For more detailed information, check out our [project page](https://ap229997.github.io/projects/transfuser), [paper](https://arxiv.org/pdf/2104.09224.pdf) and [code](https://github.com/autonomousvision/transfuser). We are happy to receive your feedback!

    @inproceedings{Prakash2021CVPR,
        title = {Multi-Modal Fusion Transformer for End-to-End Autonomous Driving},
        author = {Prakash, Aditya and Chitta, Kashyap and Geiger, Andreas},
        booktitle = {Proceedings IEEE Conf. on Computer Vision and Pattern Recognition (CVPR)},
        year = {2021}
    }