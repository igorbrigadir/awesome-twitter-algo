# Awesome Twitter Algo :bird:

Curated by [Igor Brigadir](https://github.com/igorbrigadir) and [Vicki Boykis](https://github.com/veekaybee).

An annotated look through the release of the Twitter algorithm, through the context of engineering and recsys, with notes from repo creators on significance of specific parts of the code. Since it can be hard to parse through so much code and derive meaning and context, we do it for you!

This code focuses on the services used to build the Home timeline `For You` feed, the algorithmic tab that is now served first on both web and mobile next to the   `Following` feed. 

<img width="591" alt="Screenshot 2023-03-31 at 9 36 04 PM" src="https://user-images.githubusercontent.com/3837836/229259504-fd08c5f5-a346-4e6a-b7d0-2f5514e02915.png">

# Contributing

We're happy to take changes that add and contextualize Twitter's recommendations algorithm as it's been released over the past week. To contribute, please submit a PR with good formatting and grammar and lots of links to references where relevant. We're especially happy for feedback from tweeps or former tweeps who can tell us where we got it wrong. 

 # High-level Context and Summary
One thing that's immediately obvious is that this is not the entire codebase or even a working majority of it. Missing from this codebase are 1) many flows that process,enrich, and refine model input data
2) YAML configuration metafiles which could tell us quite a bit about how the code actually works. There are only 7 of them, the rest have been redacted.
3) Most code related to spinning up the actual infrastructure
4) Git commit history that shows us how some of this code has evolved
5) A large portion of the trust and safety codebase. 

An important high-level concept discussed in the Spaces releasing this code was in-network and out-of-network. In-network tweets are those from people you follow, out-of-network is everyone else. A blend of 50%/50% are offered in the daily ~1500 tweets run through rankers. 

# Code Links

What was released? The majority of the code and algorithms, but not the data or parameters or configurations or build tools of the Recommneder Systems behind "For You" timeline recommendations. The Candidate Retrieval code was also not released, and neither was the Trust and Safety components, and the Ads components - those remain closed off. No User Data or credentials were inside the repositories and code comments were sanitized (or at least, none were obviously there on first look).

[Twitter Algo Repo](https://github.com/twitter/the-algorithm) || [Twitter ML Algo Repo](https://github.com/twitter/the-algorithm-ml) || [Blog Post](https://blog.twitter.com/engineering/en_us/topics/open-source/2023/twitter-recommendation-algorithm)

# Architecture Diagram!


![twitter architecture@2x (1)](https://user-images.githubusercontent.com/3837836/229310537-620628de-7a81-4ced-b34f-94eb3f3e2370.png)



[Link to update here.](https://whimsical.com/twitter-archtecture-PoR7TJb1eac2UofLVSY28e)

 # Programming Languages
 
 The released code comes in a variety of languages. The most common languages used at Twitter are: 
 
 + Java, used in Lucene for search indexing
 + Scala, particularly several computational frameworks including Scalding and Scio for parallel cluster computing computations
 + [Thrift](https://thrift.apache.org/), a cross-platform framework for RPC calls originally developed at Facebook(Meta)
 + Python for the machine learning models in the stack, includes both PyTorch and Tensorflow (legacy) code
 
# Recsys

![recsys](https://user-images.githubusercontent.com/3837836/229260535-27c3bcc6-403b-4d71-b301-f381b0b1be33.png)


The typical recommender system pipeline has four steps: candidate generation, ranking, filtering, and serving. 

+ Candidate generation occurs when you have millions or billions of potential items in your source data based on user-item interactions. This piece usually includes collaborative filtering or neural algorithms to reduce the size of the candiate dataset computtationally. 
+  These need to then be ranked against each other, filtered against business logic and blended and served to the user in some kind of surface area, in our case the For You feed. 

## Input Data

+ The system starts with [500 million tweets posted on a daily basis](https://blog.twitter.com/engineering/en_us/topics/open-source/2023/twitter-recommendation-algorithm). The [input data](https://blog.twitter.com/engineering/en_us/topics/infrastructure/2021/processing-billions-of-events-in-real-time-at-twitter-) is processed in this manner. 

<img width="841" alt="Screenshot 2023-04-01 at 3 26 24 PM" src="https://user-images.githubusercontent.com/3837836/229310405-a4839079-bb77-427e-bfe8-4cebd1d1a6af.png">

+ Streaming Dataflow jobs to apply deduping
+ Perform real-time aggregation and sink data into BigTable


It filters to showing you one of 1500 possible tweet generated candidates. 


## Candidate Generators

+ The largest candidate generator is [Earlybird](https://blog.twitter.com/engineering/en_us/a/2011/the-engineering-behind-twitter-s-new-search-experience), a Lucene based real-time retrieval engine. There is an [Earlybird paper.](http://notes.stephenholiday.com/Earlybird.pdf) 

## Rankers

+ **Recap** The "Heavy Ranker" is a [parallel masknet](https://arxiv.org/abs/2102.07619). Majority of the code for this is in the [ML repo](https://github.com/twitter/the-algorithm-ml/blob/main/projects/home/recap/README.md). The [ranker itself](https://github.com/twitter/the-algorithm-ml/blob/main/projects/home/recap/README.md) is run after the candidate generators. 

[Input features](https://github.com/twitter/the-algorithm-ml/blob/main/projects/home/recap/FEATURES.md) 

Ouptus are predictions on how user will respond to the tweet: 
+ probability the user will favorite the Tweet
+ probability the user will click into the conversation of this tweet and reply or like a Tweet
+ probability the user will click into the conversation of this Tweet and stay there for at least 2 minutes.
+probability the user will react negatively (requesting "show less often" on the Tweet or author, block or mute the Tweet author) 
+ probability the user opens the Tweet author profile and Likes or replies to a Tweet
+ probability the user replies to the Tweet
+ probability the user replies to the Tweet and this reply is engaged by the Tweet author 
+ probability the user will click Report Tweet 
+ probability the user will ReTweet the Tweet
+ probability (for a video Tweet) that the user will watch at least half of the video

All of these are combined and weighted into a score. Hyperparameters for the model [and weighting are here.](https://github.com/twitter/the-algorithm-ml/blob/78c3235eee5b4e111ccacb7d48e80eca019e480c/projects/home/recap/config/local_prod.yaml#L1)

+ The Light Ranker


## Filters

+ Coming into this stage from the light ranker, there are other heuristics that are [used to filter out more tweets after scoring.](https://github.com/twitter/the-algorithm-ml/blob/main/projects/home/recap/README.md)

+ Remove [out-of-network competitor site URLs](https://github.com/twitter/the-algorithm/blob/main/home-mixer/server/src/main/scala/com/twitter/home_mixer/functional_component/filter/OutOfNetworkCompetitorURLFilter.scala) from potential offered candidate Tweets

## Timeline Mixer

+ The timeline mixer has a [ratio where](https://github.com/twitter/the-algorithm/blob/7f90d0ca342b928b479b512ec51ac2c3821f5922/home-mixer/server/src/main/scala/com/twitter/home_mixer/param/HomeGlobalParams.scala#L89) verified blue checkmark tweets are offered twice as more if they're out-of-network and four times as more if they're in-network. 


## Business Terms and Logic

These are Twitter specific terms and names that keep coming up across different code bases and blog posts.

+ Twepoch - A "magic number" `1288834974657L`, which is a timestamp for `2010-11-04T01:42:54Z` the date that Twitter introduced the Snowflake ID system, used as Twitter's own ["Unix Epoch"](https://en.wikipedia.org/wiki/Epoch_(computing))
+ Snowflake - Twitter's system for [assigning unique IDs](https://blog.twitter.com/engineering/en_us/a/2010/announcing-snowflake) to tweets, users, lists, DMs, media etc. 
+ WTF - [Who to follow](https://web.stanford.edu/~rezab/papers/wtf_overview.pdf)
+ DDG - Duck Duck Goose, Twitter's [A/B Testing Platform](https://blog.twitter.com/engineering/en_us/a/2015/twitter-experimentation-technical-overview).
+ Earlybird - Twitter's Lucene based real-time search index. [Notes](https://stephenholiday.com/notes/earlybird/) and a [blog post here](https://blog.twitter.com/engineering/en_us/a/2011/the-engineering-behind-twitter-s-new-search-experience).
+ "Unregretted user minutes" - the metric Twitter publicly states is the thing they are optimizing for. It is unknown how exactly they measure this.

## Changes

+ 2 hours after it was released, [Twitter removed](https://github.com/twitter/the-algorithm/commit/ec83d01dcaebf369444d75ed04b3625a0a645eb9) feature flags that specifically higlighted Elon's account

## Resources for Learning More about Recsys

+ @karlhigley's [blog](https://practicalrecs.com/the-rest-of-the-owl.html) and [thread of threads](https://twitter.com/karlhigley/status/1138122648934916097) are very accessible things about Recommender Systems in practice.

+ A good Recommender Systems entry point is the Google Machine Learning for [Recommender Systems course](https://developers.google.com/machine-learning/recommendation), it also has a good [glossary](https://developers.google.com/machine-learning/glossary/recsystems) of terms.

+ The biggest academic recsys community is [ACM Recsys](https://recsys.acm.org/) and state-of-the-art recommender systems research is usually openly available in the [proceedings](https://dl.acm.org/conference/recsys). A lot of the presentations are on [youtube](https://www.youtube.com/@acmrecsys/videos).

+ Admittedly out of date, but still useful, is the [RecSys Wiki](https://www.recsyswiki.com/wiki/Main_Page).

+ The latest edition of the [Recommender Systems Handbook](https://link.springer.com/book/10.1007/978-1-0716-2197-4) is also a good book that covers the field well.
