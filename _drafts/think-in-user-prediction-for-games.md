---
layout: post
title: Can We Benfit Temporal Features in Games for User Behaviors Prediciton?
categories: [Machine Learning, Deep Learning]
---



For games, especially in free-to-play online games, it's very important to predict users' behaviors, such as their play-time trends, shopping cart preference, or churn points and so on.  Most online games need servers to sychronize users information, so it's very easy to record, store and process user behavior data on server side, which needs sdk to record logs, data warehouse to store data and big-data suite to process information.  In this blog, I will share a series of models to show the effectiveness of temporal features in games. 

The task is to predict whether user will churn 7 days after registering, which is a binary classification problem. The data  are user logs, distributed in decades of tables, such as logout table, purchase table, play-round table , etc . We use AUC to evaluate the effectiveness of models.

As we want to make use of temporal datas, the first model is a simple Binary GRU model.







