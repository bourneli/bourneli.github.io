---
layout: post
title: Impressive Photos in iPhone
categories: [CV]
---



I have been using iPhones since 2014, from iPhone 4s, 6, and XR to 12 present. Last year, when I was working in Singapore. I found that Photos, a build-in app in iOS, made a video with my perosnal photos and short videos. I was touched by this video, as the topic of the video is my son, White. It traces the growth of White over the years, from six months to 4 years old.

Recently, I found [an article](https://machinelearning.apple.com/research/recognizing-people-photos) related to this topic, published by the Apple technology team. This technology moves me, so I am interested in what happens behind the scene.

The main idea is face and upper body detection, image embedding, and embedding clustering.

You would know how it works form the architecture below.
![](/img/photos_in_iphone.jpeg)

From the product perspective, there are several key features. Recognize the people automatically. Let users themselves edit the name of each person. Provide image retrieval based on name. Construct a personal knowledge graph connecting people, places, and times. With the information mentioned above, the iPhone can produce videos according to the pre-designed topics, such as growths, trips, parties, etc.

Although the theory is not complicated from the computer science scene, there are many details needed to be pointed out. Privacy should be noticed, as the photos are owned by the users but not Apple, which means that most of the computing process should occur on the device side but not the server. The algorithm should perform well in all skins, sexes, and religions. Use incremental clustering to reduce the computing cost. Upper body embedding could increase the stability of clustering. Knowledge graph would be so powerful in the personal information management. MobileNet reduces the size of the model, meanwhile achieve a high precision performance.

The key efforts are paid to executing the automatic people recognization, which severely depends on the neural networks and machine learning. Technology improvements change your life, especially on the development of Artificial Intelligence. The experience of iPhone Photos is joyful. Apple is deserved to be a great company!

