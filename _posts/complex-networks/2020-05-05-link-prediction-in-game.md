---
layout: post
title:  Summary of Link Prediction in Online Games
categories: [SNS]
---



This is my first blog in English, feeling excited and nervous. I am going to communicate with people in English at work, so writing English blog is a good way to prepare for it.

I participated in the work of online game link prediction about three years ago, and I think it's time to give a summary about this work now. Generally speaking, link prediction  is to find friends for players, which is especially important for the genres of MOBA and MMORPG, such as League of Legends, Arena Of Valor, Magic Blade and so on. In our method, we divide the problem into two subproblems, finding acquaintances and finding strangers. 



## Finding Acquaintances

The players' friend lists are stored in our data warehourse. We can transform the list to a social network, in which we can recommend the 2-hop and 3-hop neighbors as acquaintances. Then, we need to sort them to create the final top-N list. The metrics for sorting are the relation similarities. We have tried the following ones:

* **Common Neighbors**  $$ \text{Sim}_{\text{CN}}(x,y) = \vert \Gamma(x) \cap \Gamma(y) \vert $$ 
* **Jaccard Similarity Coefficient**  $$ \text{Sim}_{\text{Jaccard}}(x,y) =  \text{Sim}_{\text{CN}}(x,y) / \vert \Gamma(x) \cup \Gamma(y) \vert  $$
* **Adamic-Adar**  $$ \text{Sim}_{\text{AA}}(x, y)=\sum_{z \in \Gamma(x) \cap \Gamma(y)} \vert \Gamma(z) \vert^{-1} $$ 
* **Resource Allocation**  $$ \text{Sim}_{\text{RA}}(x, y) = \sum_{z \in \Gamma(x) \cap \Gamma(y)} \log\vert \Gamma(z) \vert ^{-1} $$ 
* **Preferential Attachment**  $$ \text{Sim}_{\text{PA}}(x, y)=\vert \Gamma(x) \vert \times \vert \Gamma(y) \vert $$
* **Relation Transfer**  $$ \text{Sim}_{\text{RT}}(x, y) = \sum_{z \in \Gamma(x) \cap \Gamma(y)}  (w(x,z)w(z,y))/(w(z)w(y)) $$

$\Gamma(x)$ means the set of 1-hop neighbors of user $x$. $w (x,z)$ indicates the weight of the relation of user $z$ to $x$, so it derives $ w(x) = \sum_{n \in \Gamma(x)}w(x, n) $. 

In my opinion, these metrics are heuristic, so there is no mathematics to ensure which one is best. However, they are simple to implement. So, we can get them all to take online AB Test and choose the best one in our cases. 



## Finding Strangers

The hardest problem for the method mentioned above is cold-start. If someone hasn't had friends yet, you cannot make the recommending. In these cases, we have developed the method to recommend strangers. We uses the algorithm Locality Sensitive Hashing, LSH for short, to acquire the appropriately similar friends for each user. 

In our cases, even if the user has no friends in the game, we still can collect the user in-game actions. However, there are great number of players, so it's impossible to iterator all the other players to calculate similarities for each user. LSH can do this in an appropriate way, which indicates there are mathematics guarantees to ensure it can get similar players in high probability and , what's more, in linear time complexity. 

The following image shows how it works.

![](\img\lsh-opt-demo\lsh_demo.png)

The LSH can push the similar players into the same bucket. In each bucket, the number of user is relative small, so we can calculate the similar friends by brute force. Even if there are many users in one bucket, which we cannot by brute force, it makes sure that they are very similar with each other. As a result, we can randomly choose top-N similar users in the bucket for each one.



## Hybrid Method

We developed a more advanced hybrid method to improve the friend recommendation. We use the acquaintances and strangers as potential candidates, also known as recalling friends, and then use machine learning to train models to predict the probability of player x and y becoming friends with each other. Once you introduce the problem to machine learning, you can use many classifiers, such as logistic regression, factorization machine, random forest, gradient boosted decision tree and even the powerful deep learning models. 



## Summary

I have left the social network mining team for 3 years, and I miss these guys. They definitely have developed many new methods for link prediction recently. The article records what I was doing during the period, hoping it will be useful for the readers.  

