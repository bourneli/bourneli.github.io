---
layout: post
title: How does our AI evolve in deconstructing short videos? 
categories: [CV, Video, Tagging, Segmentation, UA, Advertisement]
---



Since 4G era, the network bandwidth increases in a huge lift. As a result, it makes short video more and more popular around the world, especially driven by Tiktok. Meanwhile, more and more advertisements appear in the form of video. According to statistics, nearly more than 80% advertisements around the global markets are videos.  

As the trend goes on, it's valuable to analyse these videos in an efficient and effective way to produce insights, so that marketing team can make use of them to acquire more new users with the same budgets. We have been working on this technology since Dec. 2021 and our solution has been evolving. So, I am going to record the main process as a memo.

Our solution has gone through two major changes, so I use two sections to present them respectively.

## State 1

From Dec. 2020 to Jun. 2021, we thought this problem as an extension of image classification. We trained many CNN-based models to classify different topics, such as terrain types, weapons, carriers, etc. We sampled frames from each video, and used these models to obtain the tags for each frame. Then, we use rule-based method to aggregate the tags to obtain higher level tags. 

The method works, but not elegantly, and even a bit of uglily. First, we should maintain many CNN-based models, which required many resource to maintain them. Then, we did not leverage the background music, roles' speeches and text in the video, which means we had wasted a lot of information in the videos.

At April, TAAC(Tencent Advertisement Algorithm Competition) 2021 began. The topic of it is to deconstruct the advertisement, which first segments a video temporally and then tags each part. As a result, we can get the structure of a video, with which it can be very easy to make deeper insights. All of it is what we purse, so we participated the competition. After 2 months hard work, we had learned and invested so much in it, but it deserved that our team won the internal 2rd prize finally.



## State 2

From Jul. 2021 to present, our team began to refactor our first stage model with the approach learned from the competition. We treat it as a multi-modal problem, which uses deep learning models to extract the information from video, audio, texts. Then, aggregate these information to segment and tag the video. There are two end-to-end tasks: Segmentation and Tagging, and the technology architecture is following:

![](/img/short-video-structuring-roadmap/state2-arch.png) 

It uses three main deep learning models to process videos, audio and text respectively and extract features, then the segmentation net takes the video features and calculates the segments, and the tagging net classifies tags in each segment. The following list is the main algorithms that we have used or will try in the future.

* CV Backbone: Res-Net , Efficient Net, Botong(in-house)
* Audio Backbone: VGGish
* NLP Backbone: BERT
* Segmentation Net: TCN，LGSS，RepNet，TransNet, unet ++, BMN
* Tagging Net: Swin, clip transfomer, VIT 

As you can see, the architecture is simple, but there are so many elements models that we can composite to get the best performance. There are so many details, both in algorithm level and engineer level, that I ignored, in order to present the whole picture in the limited space. 

OK, that is all, hoping you enjoy it!
