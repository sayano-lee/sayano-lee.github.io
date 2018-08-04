---
title: 'Paper Read: Dense-Captioning Events in Videos'
date: 2018-07-31 22:16:46
tags: 
	- cv 
	- nlp
	- video captioning
	- iccv
---

Dense-Captioning Events in Videos [arxiv](https://arxiv.org/abs/1705.00754)
Author: Ranjay Krishna, Kenji Hata, Frederic Ren, Li Fei-Fei, Juan Carlos Niebles
Affiliation: Stanford University
Publication: ICCV2017
Github: [website](https://github.com/ranjaykrishna/densevid_eval) ***only eval codes provided***
project page: [website](https://cs.stanford.edu/people/ranjaykrishna/densevid/)

<!-- more -->

## Introduction

The author observed that conventional video captioning models were not able to (1) *capture details* or to (2) *identify all events in a video*. Therefore, the author proposed a novel video captioning task called Dense-Captioning Events and a corresponding dataset, ActivityNet captions dataset, to tackle those problems. The author also mentioned that Dense-Captioning events is analogous to dense-image-captioning, and difference between videos and images is difference between time and space. To be specific, video captioning requires temporal localization and description, while image captioning requires spatial localization and description.

Therefore, the newly-proposed events task requires detecting as many as events in a video segment and generating appropriate captions for each identified event. 

Two key challenges are proposed: 
(1) events in given videos can range across multiple time scales and can even overlap (related to proposals)
(2) events in given videos are usually related to one another (related to captioning)

Given the first challenge, one single proposal may contain several video clips, whose features are extracted in advance, and the proposal features are either encoded by mean pooling <sup>50</sup> or RNN <sup>49</sup> in conventional methods, which cannot deal with long video segments because of vanishing gradients. Therefore, the author proposed to extend [DAPS]() to multi-scale detection of events.

Given the second challenge, according to the author, there are not many existed video captioning tasks with multiple captions. An example was given about "cooking" videos in which events happened sequentially and highly correlated to the objects in the video <sup>37</sup>. It is not a representative model for "open" domain videos, thus the author utilizes the context from all past and future ***events*** (proposals).

## Methodology

### Models

![ActivityNet](model.jpg)

Main components of the proposed pipeline are Proposal module and Captioning module. As described in Introduction, the Proposal module is able to encode video features into multiple scales to generate several proposals. 

The goal of this pipeline is to jointly localize temporal proposals and describe each of them with natural language. As mentioned before, this pipeline is designed to deal with two key challenges: events overlapping and interconnection between events. The author proposed proposal module and captioning module to handle them respectively. Specifically, the *Proposal module* can encode multi-scale video clips to mitigate long-time video problems as well as overlapping problems, and *Captioning module* takes contexts between past and future into consideration, extracting intersectional infos from all proposal events. 

#### Proposal Module
<!-- The Proposal module is a modification from [DAPS](), and DAPS pipelines are as follows: -->

<!-- ![DAPs](daps_model.jpg) -->

![Proposal Module of ActivityNet](proposal_model.jpg)
The architecture of this proposal module is able to tackle the challenge of detecting events in short as well as long video sequences, while preventing the dense application of the language model over sliding windows during inference. The input of the Proposal module is a series of semantic features extracted from video frames whose total length is T:

{% blockquote %}
{f = F(v<sub>t</sub> : v<sub>t+δ</sub>)}  <sub>δ = 16</sub>
{% endblockquote %}

$$\begin{equation}
e=mc^2
\end{equation}\label{eq1}$$

$\eqref{eq1}$

The output of the function F is a matrix with size of N x D, where N equals T/δ and D equals 500, which means neighboring semantic features (green features) are extracted from non-overlapping video clips.

Video features are then sent into a LSTM proposal architecture to generate event proposals, with strides 1, 2, 4 and 8 as showed in Proposal module (red part) <sup>[?](faster rcnn anchors)</sup>. Different strides are calculated in parallel in order to decrease iteration times on video features. When proposal LSTM detects an event, the hidden state of proposal LSTM at that time step is chosen to be the proposal feature. <sup>[?](start?end? or fixed length?)</sup> Obviously, overlapped event proposals will be generated in this process. The original DAPS uses non-maximum suppression to eliminate overlapping events, while ActivityNet keeps them separately and treat them as individual events. 

#### Captioning Module

![Captioning Module of ActivityNet](captioning_model.jpg)