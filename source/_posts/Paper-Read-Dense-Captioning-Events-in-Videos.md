---
title: 'Paper Read: Dense-Captioning Events in Videos'
date: 2018-07-31 22:16:46
tags:
---

Dense-Captioning Events in Videos [arxiv](https://arxiv.org/abs/1705.00754)
Author: Ranjay Krishna, Kenji Hata, Frederic Ren, Li Fei-Fei, Juan Carlos Niebles
Affiliation: Stanford University
Publication: ICCV2017
Github: [website](https://github.com/ranjaykrishna/densevid_eval) ***only eval codes provided***
project page: [website](https://cs.stanford.edu/people/ranjaykrishna/densevid/)

## Problem Formulation

The author observed that conventional video captioning models were not able to (1) *capture details* or to (2) *identify all events in a video*. Therefore, the author proposed a novel video captioning task called Dense-Captioning Events and a corresponding dataset, ActivityNet captions dataset, to tackle those problems. The author also mentioned that Dense-Captioning events is analogous to dense-image-captioning, and difference between videos and images is difference between time and space. To be specific, video captioning requires temporal localization and description, while image captioning requires spatial localization and description.

This newly-proposed events task requires detecting as many as events in a video segment and generating appropriate captions for each identified event. 

Two key challenges proposed: 
(1) events in given videos can range across multiple time scales and can even overlap (related to proposals)
(2) events in given videos are usually related to one another (related to captioning)

Given the first challenge, one single proposal may contain several video clips, whose features are extracted in advance, and the proposal features are either encoded by mean pooling <sup>50</sup> or RNN <sup>49</sup> in conventional methods, which cannot deal with long video segments because of vanishing gradients. Therefore, the author proposed to extend [DAPS]() to multi-scale detection of events.

Given the second challenge, according to the author, there are not many existed video captioning tasks with multiple captions. An example was given about "cooking" videos in which events happened sequentially and highly correlated to the objects in the video <sup>37</sup>. It is not a representative model for "open" domain videos, thus the author utilizes the context from all the events 