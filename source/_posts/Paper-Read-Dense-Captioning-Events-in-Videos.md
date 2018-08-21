---
title: 'Paper Read: Dense-Captioning Events in Videos'
date: 2018-07-31 22:16:46
tags: 
	- cv 
	- nlp
	- video captioning
	- iccv2017
---

<div id="title">
	<span>Dense-Captioning Events in Videos</span> 
	<a href="https://arxiv.org/abs/1705.00754">arxiv</a>
</div>

<ul class="aff">
	<li>Author: Ranjay Krishna, Kenji Hata, Frederic Ren, Li Fei-Fei, Juan Carlos Niebles</li>
	<li>Affiliation:
		<ul class="aff2">
			<li>Stanford University</li>
		</ul>		
	</li>
	<li>Publication: ICCV2017</li>
	<li>Github: <a href="https://github.com/ranjaykrishna/densevid_eval">website</a> <span class='notes'>(only eval codes provided)</span></li>
	<li>Project Page: <a href="https://cs.stanford.edu/people/ranjaykrishna/densevid/">website</a></li>
</ul>

<!-- more -->

## Introduction

<!-- Why 
	
	Problems existed in conventional tasks in the similar field

-->

The author observed that conventional video captioning models were not able to (1) *capture details* or to (2) *identify all events in a video*. Therefore, the author proposed a novel video captioning task called Dense-Captioning Events and a corresponding dataset, ActivityNet captions dataset, to tackle those problems. The author also mentioned that Dense-Captioning events is analogous to dense-image-captioning, and difference between videos and images is difference between time and space. To be specific, video captioning requires temporal localization and description, while image captioning requires spatial localization and description.

<!-- What 

	Methods proposed to deal with the aforementioned problems

-->

Therefore, the newly-proposed events task requires detecting as many as events in a video segment and generating appropriate captions for each identified event. 

<!-- How 

	Specific explanations for the methods proposed

-->

Two key challenges are proposed: 
(1) events in given videos can range across multiple time scales and can even overlap (related to proposals)
(2) events in given videos are usually related to one another (related to captioning)

Given the first challenge, one single proposal may contain several video clips, whose features are extracted in advance, and the proposal features are either encoded by mean pooling <sup>[1](#1)</sup> or RNN <sup>[2](#2)</sup> in conventional methods, which cannot deal with long video segments because of vanishing gradients. Therefore, the author proposed to extend DAPS <sup>[3](#3)</sup> to multi-scale detection of events.

Given the second challenge, according to the author, there are not many existed video captioning tasks with multiple captions. An example was given about "cooking" videos in which events happened sequentially and highly correlated to the objects in the video <sup>[4](#4)</sup>. It is not a representative model for "open" domain videos, thus the author utilizes the context from all past and future ***events*** (proposals).


## Models

![ActivityNet](model.jpg)

Main components of the proposed pipeline are Proposal module and Captioning module. As described in Introduction, the Proposal module is able to encode video features into multiple scales to generate several proposals. 

The goal of this pipeline is to jointly localize temporal proposals and describe each of them with natural language. As mentioned before, this pipeline is designed to deal with two key challenges: events overlapping and interconnection between events. The author proposed proposal module and captioning module to handle them respectively. Specifically, the *Proposal module* can encode multi-scale video clips to mitigate long-time video problems as well as overlapping problems, and *Captioning module* takes contexts between past and future into consideration, extracting intersectional infos from all proposal events. 

### Proposal Module

![Proposal Module of ActivityNet](proposal_model.jpg)
The architecture of this proposal module is able to tackle the challenge of detecting events in short as well as long video sequences, while preventing the dense application of the language model over sliding windows during inference. The input of the Proposal module is a series of semantic features extracted from video frames whose total length is T: 

$$
f_t = F(v_t : v_{t+\sigma})  \hspace{2em} \sigma = 16
$$

The output of the function F is a matrix with size of N x D, where N equals $\frac{T}{δ}$ and D equals 500, which means neighboring semantic features (green features) are extracted from non-overlapping video clips.

Video features are then sent into a LSTM proposal architecture to generate event proposals, with strides 1, 2, 4 and 8 as showed in Proposal module (red part) <sup>[?](faster rcnn anchors)</sup>. Different strides are calculated in parallel in order to decrease iteration times on video features. When proposal LSTM detects an event, the hidden state of proposal LSTM at that time step is chosen to be the proposal feature. <sup>[?](start?end? or fixed length?)</sup> Obviously, overlapped event proposals will be generated in this process. The original DAPS uses non-maximum suppression to eliminate overlapping events, while ActivityNet keeps them separately and treat them as individual events. 

### Captioning Module

![Captioning Module of ActivityNet](captioning_model.jpg)
The captioning module is able to capture all events from past to future. And attention mechanism is taken into consideration to integrate both past and future event proposal features. The past and future context representations are calculated as follows:

$$
h_{i}^{past} = \frac{1}{Z^{past}}\sum_{j \neq i} \mathbb{1} [t_{j}^{end} \lt t_{i}^{end}] \omega_{j}h_{j}
$$
$$
h_{i}^{future} = \frac{1}{Z^{future}}\sum_{j \neq i} \mathbb{1} [t_{j}^{end} \geqq t_{i}^{end}] \omega_{j}h_{j}
$$

And:

$$
Z^{past} = \sum_{j \neq i} \mathbb{1} [t_{j}^{end} \lt t_{i}^{end}] \hspace{2em} Z^{future} = \sum_{j \neq i} \mathbb{1} [t_{j}^{end} \geqq t_{i}^{end}]
$$

$\omega_{j}$ represents attention-weighted event proposal features, calculated as follows:

$$
a_{i} = \omega_{a}h_{i} + b_{a}
$$
$$
\omega_{j} = a_{j}h_{j}
$$

$a_{i}$ represents attention factor, calculated from learned weight $\omega_{a}h_{i}$ and bias $b_{a}$. Then ($h_{past}$, $h_{i}$, $h_{future}$) are concatenated and fed into LSTM language model to describe events. 


## ActivityNet Datasets

The ActivityNet Captions dataset connects videos to a series of temporally annotated sentences. Each sentence covers an unique segment of the video, describing an event that occurs. These events may occur over very long or short periods of time and are not limited in any capacity, allowing them to co-occur. 

### Dataset Analysis

* each of the 20k videos in ActivityNet Captions contains 3.65 temporally localized sentences on average, resulting in a total of 100k sentences
* each sentence has an average length of 13.48 words, which is also normally distributed
* each sentence describes 36 seconds and 31% of their respective videos on average
* the entire paragraph for each video on average describes 94.6% of the entire video
* 10% of the temporal descriptions overlap, showing that the events cover simultaneous events

With the percentage of verbs comprising ActivityNet Captions being significantly more, the author finds that ActivityNet Captions shifts sentence descriptions from being object-centric in images to action-centric in videos. 

## Summary

This paper proposes a new captioning task which describes an arbitrary-length video segment with several events, and a method called ActivityNet Captioning model to deal with this challenge. The proposed pipeline contains two parts: the Proposal module and the Captioning module. The Proposal module generates event proposals with overlapping from pre-extracted video features. The Captioning module generates corresponding events from attention-weighted contextual features between past and future. 

## References

<span id="1" class="ref">[1] S. Venugopalan, H. Xu, J. Donahue, M. Rohrbach, R. Mooney, and K. Saenko. Translating videos to natural language using deep recurrent neural networks. arXiv preprint arXiv:1412.4729, 2014.</span>
<span id="2" class="ref">[2] S. Venugopalan, M. Rohrbach, J. Donahue,  R. Mooney, T. Darrell, and K. Saenko. Sequence to sequence video to text.  In Proceedings of the IEEE International Conference on Computer Vision, pages 4534–4542, 2015.</span>
<span id="3" class="ref">[3] V. Escorcia, F. C. Heilbron, J. C. Niebles, and B. Ghanem. Daps:Deep action proposals for action understanding. In European Conference on Computer Vision, pages 768–784. Springer, 2016.</span>
<span id="4" class="ref">[4] A. Rohrbach, M. Rohrbach, W. Qiu, A. Friedrich, M. Pinkal, and B. Schiele. Coherent multi-sentence video description with variable level of detail.  In German Conference on Pattern Recognition, pages 184–195. Springer, 2014.</span>


<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.10.0-beta/dist/katex.min.css" integrity="sha384-9tPv11A+glH/on/wEu99NVwDPwkMQESOocs/ZGXPoIiLE8MU/qkqUcZ3zzL+6DuH" crossorigin="anonymous">
<script src="https://cdn.jsdelivr.net/npm/katex@0.10.0-beta/dist/katex.min.js" integrity="sha384-U8Vrjwb8fuHMt6ewaCy8uqeUXv4oitYACKdB0VziCerzt011iQ/0TqlSlv8MReCm" crossorigin="anonymous"></script>
<script src="https://cdn.jsdelivr.net/npm/katex@0.10.0-beta/dist/contrib/auto-render.min.js" integrity="sha384-aGfk5kvhIq5x1x5YdvCp4upKZYnA8ckafviDpmWEKp4afOZEqOli7gqSnh8I6enH" crossorigin="anonymous"></script>

<script>renderMathInElement(document.body);</script>