---
layout: single
title:  "Label Efficient Visual Abstractions for Autonomous Driving"
date:   2020-10-05 07:15 +0200
categories: "paper"
tags: ["self-driving", "autonomous vehicles", "imitation learning", "semantic segmentation", "object detection"]
author: "Kashyap Chitta"
excerpt: >-
    We analyze the trade-off between annotation time & driving policy performance for several intermediate scene representations.
header:
    teaser: "/assets/posts/2020-10-05-visual-abstractions/teaser.png"
---

Recent Artificial Intelligence (AI) systems have achieved impressive feats. Human world champions were convincingly defeated by AI agents that learn *policies* to play the [board game Go](https://deepmind.com/research/case-studies/alphago-the-story-so-far) as well as video games [Starcraft II](https://deepmind.com/blog/article/alphastar-mastering-real-time-strategy-game-starcraft-ii) and [Dota 2](https://openai.com/blog/openai-five/). These policies map *observations of the game state* to actions using a Deep Neural Network (DNN). However, AI systems designed to operate in the physical word, such as self-driving vehicles, do not observe a concise game state summarizing their surrounding environment. Typically, these systems initially perform *perception*, processing information from sensors (like a camera or LiDAR) into a *scene representation*, which acts like a game state. They then perform *control*, where this scene representation is input to a policy that outputs an action (steering angle, throttle and brake values).

![overview]({{ site.url }}/assets/posts/2020-10-05-visual-abstractions/overview.png){: .align-center}

## What is a Good Scene Representation?

A good scene representation should contain *only* the information relevant to the driving task, with as little additional information as possible. One example of a scene representation is a semantic segmentation map, which classifies each image pixel into a category (such as road, lane marking, pedestrian, or vehicle).

![teaser_gt_14]({{ site.url }}/assets/posts/2020-10-05-visual-abstractions/teaser_gt_14.png){: .align-center}

[Previous work](http://vladlen.info/papers/does-vision-matter.pdf) has shown that semantic segmentation is an effective scene representation for urban autonomous driving. However, obtaining a training dataset with accurate annotations for semantic segmentation is expensive. For instance, labeling a single image was reported to take 90 minutes on average for the [Cityscapes dataset](https://www.cityscapes-dataset.com/). So far, there has been no systematic study on label efficiency when using semantic segmentation maps as scene representations. Therefore, in our recent work [Label Efficient Visual Abstractions for Autonomous Driving](http://www.cvlibs.net/publications/Behl2020IROS.pdf), we look into the following question - *can scene representations obtained from datasets with lower annotation costs be competitive in terms of driving ability?*

## Our Study

The two key ideas we leverage to improve label efficiency are:
1. Considering only safety-critical semantic categories and combining several non-salient classes (e.g., sidewalk and building) into a single category.
2. Using coarse 2D bounding boxes to represent objects such as pedestrians, vehicles and traffic lights.

![teaser_coarse]({{ site.url }}/assets/posts/2020-10-05-visual-abstractions/teaser_coarse.png){: .align-center}

## Results
We conduct extensive experiments on the [CARLA NoCrash](https://arxiv.org/pdf/1904.08980.pdf) benchmark. One of our key findings is that limiting the representation to six semantic classes (road, lane markings, pedestrians, vehicles, red lights and green lights), surprisingly leads to improved performance when compared to the commonly-used set of fourteen semantic classes. In our paper, we demonstrate improved driving performance while reducing the total time required to annotate the training dataset from 7500 hours to just 50 hours.

![result_1]({{ site.url }}/assets/posts/2020-10-05-visual-abstractions/result_1.gif){: .align-center}

Moreover, we observe significantly better driving performance and more consistent, reproducible results with different random seeds for training than state-of-the-art image-based methods such as [CILRS (Codevilla et al., 2019)](https://arxiv.org/pdf/1904.08980.pdf).

![result_2]({{ site.url }}/assets/posts/2020-10-05-visual-abstractions/result_2.gif){: .align-center}

## Further Information
To learn more about work, we invite you to watch our talk:

{% include video id="5SszfDWrqzo" provider="youtube" %}

More information (including the [paper](http://www.cvlibs.net/publications/Behl2020IROS.pdf) and [source code](https://github.com/autonomousvision/visual_abstractions)) is available on our [project page](https://avg.is.tuebingen.mpg.de/publications/behl2020iros).

    @inproceedings{Behl2020IROS,
        title = {Label Efficient Visual Abstractions for Autonomous Driving},
        author = {Behl, Aseem and Chitta, Kashyap and Prakash, Aditya and Ohn-Bar, Eshed and Geiger, Andreas},
        booktitle = {International Conference on Intelligent Robots and Systems (IROS)},
        year = {2020}
    }
