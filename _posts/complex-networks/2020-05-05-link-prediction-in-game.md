---
layout: post
title:  Summary of Link Prediction in Online Games
categories: [SNS]
---



This is my first blog in English, feeling excited and nervous. I am going to communicate with people in other countries at work in English, so writing English blog is a good way to get prepared for it.

I participated in the work of online game link prediction about three years ago, and I think it's time to give a summary about this work. Generally speaking, link prediction in online games is to find friends for players, which is especially important for the genre of MOBA and MMORPG, such as League of Legends, Arena Of Valor, Magic Blade and so on. In our method, we divide the problem into two subproblems, finding acquaintances and finding strangers. 



## Finding Acquaintances

The players' friend lists are stored in the data warehourse. We can convert the list to a social network, in which we can recommend the 2-hop and 3-hop neighbors as acquaintances. Then, we need to sort them to create the final top-N list. The metrics are the relation similarities. We have tried the following ones:

* **Common Neighbors** $ \text{Sim}_{\text{CN}}(x,y) = \vert \Gamma(x) \cap \Gamma(y) \vert $ 
* **Jaccard Similarity Coefficient** $ \text{Sim}_{\text{Jaccard}}(x,y) =  \text{Sim}_{\text{CN}}(x,y) / \vert \Gamma(x) \cup \Gamma(y) \vert  $
* **Adamic-Adar** $ \text{Sim}_{\text{AA}}(x, y)=\sum_{z \in \Gamma(x) \cap \Gamma(y)} \vert \Gamma(z) \vert^{-1} $ 
* **Resource Allocation** $ \text{Sim}_{\text{RA}}(x, y) = \sum_{z \in \Gamma(x) \cap \Gamma(y)} \log\vert \Gamma(z) \vert ^{-1} $ 
* **Preferential Attachment**  $ \text{Sim}_{\text{PA}}(x, y)=\vert \Gamma(x) \vert \times \vert \Gamma(y) \vert $
* **Relation Transfer**  $ \text{Sim}_{\text{RT}}(x, y) = \sum_{z \in \Gamma(x) \cap \Gamma(y)}  (w(x,z)w(z,y))/(w(z)w(y)) $



$\Gamma(x)$ means the set of 1-hop neighbors of user $x$. For user $x$, $w(x,z)$ means the relation weight of user $z$, so it derives $ w(x) = \sum_{n \in \Gamma(x)}w(x, n) $. 

In my opinion, these metrics are heuristic, which means there is no mathematics to ensure which one is best. However, they are very simple to implement. So, we can get them all to take online AB Test and choose the best one in each case. 



## Finding Strangers

The biggest problem for the method above is cold start. If someone hasn't had friends yet, you cannot make the recommending. In these cases, we have developed the method to recommend strangers. We uses locality sensitive hashing to acquire the appropriately similar friends for each user. 

In our cases, even if the user has no friend in the game, we still can collect the user in-game action. However, there are great number of players, so it's impossible to iterator all the other players to calculate similarities for each user. LSH can do this in an appropriate way, which means there are mathematics guarantees to ensure it can get similar players in high probability and , what's more, in linear time complexity. 

The following image shows how it works.

![](\img\lsh-opt-demo\lsh_demo.png)

The LSH Functions can push the similar players into the same hashing bucket. In each bucket, the number of user is relative small, so we can calculate the similar friends by brute force. Even if there are many users in one bucket, which we cannot by brute force, it makes sure that they are very similar with each other. As a result, we can randomly choose top-N similar users in the bucket for each one.



## Hybrid Method

We developed a more advanced hybrid method to improve the friend recommendation. We use the acquaintances and strangers as potential candidates, also known as recalling items, and then use machine learning to train models to predict the probability of player x and y becoming friends. So, you can use many methods, such as logistic regression, factorization machine, random forest, gradient boosted decision tree and even the complicated deep learning methods to improve the recommending effect. 



## Summary

I have left the social network mining team for 3 years, and I miss these guys. They definitely have developed many new methods for link prediction. The article records what I was doing during the period, hoping it will be useful for the readers.  

