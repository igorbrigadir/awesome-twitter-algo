# Awesome Twitter Algo :bird:

Curated by [Igor Brigadir](https://github.com/igorbrigadir) and [Vicki Boykis](https://github.com/veekaybee).


An annotated look through the release of the Twitter algorithm, through the context of engineering and recsys, with notes from repo creators on significance of specific parts of the code. Since it can be hard to parse through so much code and derive meaning and context, we do it for you!

This code focuses on the services used to build the Home timeline `For You` feed, the algorithmic tab that is now served first on both web and mobile next to the   `Following` feed. 

<img width="591" alt="Screenshot 2023-03-31 at 9 36 04 PM" src="https://user-images.githubusercontent.com/3837836/229259504-fd08c5f5-a346-4e6a-b7d0-2f5514e02915.png">

# Contributing

We're happy to take changes that add and contextualize Twitter's recommendations algorithm as it's been released over the past week. To contribute, please submit a PR with good formatting and grammar and lots of links to references where relevant. We're especially happy for feedback from tweeps or former tweeps who can tell us where we got it wrong. 

 # High-level Context
An important high-level concept discussed in the Spaces releasing this code was in-network and out-of-network. In-network tweets are those from people you follow, out-of-network is everyone else. A blend of 50%/50% are offered in the daily ~1500 tweets run through rankers. 

# Code Links
[Twitter Algo Repo](https://github.com/twitter/the-algorithm) || [Twitter ML Algo Repo](https://github.com/twitter/the-algorithm-ml) || [Blog Post](https://blog.twitter.com/engineering/en_us/topics/open-source/2023/twitter-recommendation-algorithm)

# Architecture Diagram!

![architecture](https://user-images.githubusercontent.com/3837836/229260006-ce800791-6831-4a6c-bf52-1444becb27cc.png)


[Link to update here.](https://whimsical.com/twitter-archtecture-PoR7TJb1eac2UofLVSY28e)

 # Programming Languages
 
 The released code comes in a variety of languages. The most common languages used at Twitter are: 
 
 + Java
 + Scala, particularly several computational frameworks including Scalding and Scio for parallel cluster computing computations
 + [Thrift](https://thrift.apache.org/), a cross-platform framework for RPC calls originally developed at Facebook(Meta)
 + Python for the machine learning models in the stack 
 
# Recsys

![recsys](https://user-images.githubusercontent.com/3837836/229260535-27c3bcc6-403b-4d71-b301-f381b0b1be33.png)


The typical recommender system pipeline has four steps: candidate generation, ranking, filtering, and serving. 

+ Candidate generation occurs when you have millions or billions of potential items in your source data based on user-item interactions. This piece usually includes collaborative filtering or neural algorithms to reduce the size of the candiate dataset computtationally. 
+  These need to then be ranked against each other, filtered against business logic and blended and served to the user in some kind of surface area, in our case the For You feed. 

## Input Data



+ The system starts with [500 million tweets posted on a daily basis](https://blog.twitter.com/engineering/en_us/topics/open-source/2023/twitter-recommendation-algorithm). It filters to showing you one of 1500 possible tweet generated candidates. 


## Candidate Generators


## Rankers



## Filters

+ Remove [out-of-network competitor site URLs](https://github.com/twitter/the-algorithm/blob/main/home-mixer/server/src/main/scala/com/twitter/home_mixer/functional_component/filter/OutOfNetworkCompetitorURLFilter.scala) from potential offered candidate Tweets

## Timeline Mixer

+ The timeline mixer has a [ratio where](https://github.com/twitter/the-algorithm/blob/7f90d0ca342b928b479b512ec51ac2c3821f5922/home-mixer/server/src/main/scala/com/twitter/home_mixer/param/HomeGlobalParams.scala#L89) verified blue checkmark tweets are offered twice as more if they're out-of-network and four times as more if they're in-network. 


## Business Terms and Logic

+ [Twepoch](https://github.com/twitter/the-algorithm/blob/ec83d01dcaebf369444d75ed04b3625a0a645eb9/src/java/com/twitter/search/earlybird_root/filters/ResultTierCountFilter.java#L106), `2010-11-04T01:42:54Z` is the release date of Twitter
+ WTF - [Who to follow](https://web.stanford.edu/~rezab/papers/wtf_overview.pdf)

## Changes

+ 2 hours after it was released, [Twitter removed](https://github.com/twitter/the-algorithm/commit/ec83d01dcaebf369444d75ed04b3625a0a645eb9) feature flags that specifically higlighted Elon's account

## Resources for Learning More about Recsys
