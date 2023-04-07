# Awesome Twitter Algo :bird:

Curated by [Igor Brigadir](https://github.com/igorbrigadir) and [Vicki Boykis](https://github.com/veekaybee).

An annotated look through the release of the Twitter algorithm, through the context of engineering and recsys, with notes from repo creators on significance of specific parts of the code. Since it can be hard to parse through so much code and derive meaning and context, we do it for you!

This code focuses on the services used to build the Home timeline `For You` feed, the algorithmic tab that is now served first on both web and mobile next to the   `Following` feed. 

<img width="591" alt="Screenshot 2023-03-31 at 9 36 04 PM" src="https://user-images.githubusercontent.com/3837836/229259504-fd08c5f5-a346-4e6a-b7d0-2f5514e02915.png">

# Contributing

We're happy to take changes that add and contextualize Twitter's recommendations algorithm as it's been released over the past week. To contribute, please submit a PR with good formatting and grammar and lots of links to references where relevant. We're especially happy for feedback from tweeps or former tweeps who can tell us where we got it wrong. 

 # High-level Context and Summary
One thing that's immediately obvious is that this is not the entire codebase or even a working majority of it. Missing from this codebase are 

1) many flows that process,enrich, and refine model input data
2) YAML configuration metafiles which could tell us quite a bit about how the code actually works. There are only 7 of them, the rest have been redacted.
3) Most code related to spinning up the actual infrastructure
4) Git commit history that shows us how some of this code has evolved
5) A large portion of the trust and safety codebase, which Twitter [has noted they've left out for now](https://github.com/twitter/the-algorithm/tree/main/trust_and_safety_models#trust-and-safety-models)

An important high-level concept discussed in the Spaces releasing this code was in-network and out-of-network. In-network tweets are those from people you follow, out-of-network is everyone else. A blend of 50%/50% are offered in the daily ~1500 tweets run through rankers. 

# Code Links

What was released? The majority of the code and algorithms, but not the data or parameters or configurations or build tools of the Recommneder Systems behind "For You" timeline recommendations. The Candidate Retrieval code was also not released, and neither was the Trust and Safety components, and the Ads components - those remain closed off. No User Data or credentials were inside the repositories and code comments were sanitized (or at least, none were obviously there on first look).

[Twitter Algo Repo](https://github.com/twitter/the-algorithm) || [Twitter ML Algo Repo](https://github.com/twitter/the-algorithm-ml) || [Blog Post](https://blog.twitter.com/engineering/en_us/topics/open-source/2023/twitter-recommendation-algorithm)

# Recsys Architecture Diagram

![twitter architecture@2x (1)](https://user-images.githubusercontent.com/3837836/229310537-620628de-7a81-4ced-b34f-94eb3f3e2370.png)

[Link to update here.](https://whimsical.com/twitter-archtecture-PoR7TJb1eac2UofLVSY28e)


# Twitter Data-Centric Historical Architecture Context 

There is a very, very old post from 2013 on [High Scalability](http://highscalability.com/blog/2013/7/8/the-architecture-twitter-uses-to-deal-with-150m-active-users.html) which gives some context to how these systems were initially constructed. 

As context, Twitter initially ran all workloads on-prem but has been [moving to Google Cloud.](https://cloud.google.com/blog/products/data-analytics/how-twitter-modernized-its-data-processing-with-google-cloud). In 2019, Twitter began by migrating to BigQuery and DataFlow from a data and analytics perspective. Before the move to BigQuery, [much of the data was stored in HDFS using Thrift](https://blog.twitter.com/engineering/en_us/topics/infrastructure/2022/scaling-data-access-by-moving-an-exabyte-of-data-to-google-cloud). It currently lives in BigQuery and is processed for many of the pipelines described below using [DataFlow](https://cloud.google.com/dataflow), GCP's Spark/Scalding-processing equivaent platform.  

 # Programming Languages and Frameworks
 
 The released code comes in a variety of languages. The most common languages used at Twitter are: 

 ## Java
 
 + Used in Lucene for search indexing
 + Used in the [GraphJet](https://github.com/twitter/GraphJet) library


 ## Scala
 + [Scalding](https://github.com/twitter/scalding), written at Twitter, pre-cursor to Spark
 + [Scio](https://github.com/spotify/scio) for parallel cluster computing computations

 ## Python
 + Python for the machine learning models in the stack, includes both PyTorch and Tensorflow (legacy) code

 ## Frameworks and Metalanguages

 + Bazel for [building Scala and Java services](https://bazel.build/)
 + Starlark for [bazel configuration](https://github.com/bazelbuild/starlark)
  + [Thrift](https://thrift.apache.org/), a cross-platform framework for RPC calls originally developed at Facebook(Meta)
  + [Hadoop](https://blog.twitter.com/engineering/en_us/topics/infrastructure/2023/kerberizing-hadoop-clusters-at-twitter) - Twitter still runs one of the largest installs of Hadoop out there

# Internal Libraries 

+ [Finagle](https://twitter.github.io/finagle/) is a service written in Scala with Java and Scala APIs used to manage RPCs
+ Snowflake is [a service that generates unique identifiers](https://blog.twitter.com/engineering/en_us/a/2010/announcing-snowflake) for each tweet based on timestamp, worker number, and sequence number
+ [Heron](https://blog.twitter.com/engineering/en_us/topics/open-source/2018/heron-donated-to-apache-software-foundation) - A realtime streaming analytics library (similar to Flink) 
+ [Strato](https://www.youtube.com/watch?v=E1gDNHZr1NA) - a virtual database powered by microservices 

# Recsys

![recsys](https://user-images.githubusercontent.com/3837836/229260535-27c3bcc6-403b-4d71-b301-f381b0b1be33.png)


The typical recommender system pipeline has four steps: candidate generation, ranking, filtering, and serving. Twitter has many pipelines for performing verious parts of this this across the overall released codebase. 

+ **Candidate generation** occurs when you have millions or billions of potential items in your source data based on user-item interactions. This piece usually includes collaborative filtering or neural algorithms to reduce the size of the candiate dataset for downstream tasks.  
+  These need to then be **ranked** against each other, **filtered** against business logic and blended and served to the user in a given surface area, in this case the `For You` feed in the Twitter timeline. 

## Input Data

+ The system starts with [500 million tweets posted on a daily basis](https://blog.twitter.com/engineering/en_us/topics/open-source/2023/twitter-recommendation-algorithm). 

The [input data](https://blog.twitter.com/engineering/en_us/topics/infrastructure/2021/processing-billions-of-events-in-real-time-at-twitter-) 
comes from: 

+ Kafka
+ Twitter Eventbus
+ GCS
+ Vertica
+ [Manhattan](https://blog.twitter.com/engineering/en_us/a/2014/manhattan-our-real-time-multi-tenant-distributed-database-for-twitter-scale), a real-time multitenant distributed database that was initially developed as a serving layer on top of Hadoop and includes both observability and other metrics.


<img width="841" alt="Screenshot 2023-04-01 at 3 26 24 PM" src="https://user-images.githubusercontent.com/3837836/229310405-a4839079-bb77-427e-bfe8-4cebd1d1a6af.png">

In migrating to GCP, the current data ingest looks something like this: 
+ Streaming Dataflow jobs to apply deduping
+ Perform real-time aggregation and sink data into BigTable

That data is then made available to the candidate generation phase. There is not much about the actual data, even what a schema might look like, in the repo. 

## Candidate Generators 
(also called "features" in the chart)

### GraphJet
--- 
+ [GraphJet](https://github.com/twitter/GraphJet) - A realtime Java graph processing library that allows for in-memory processing on a single server and focuses on providing content recommendations. [Paper here.](http://www.vldb.org/pvldb/vol9/p1281-sharma.pdf) Recommendations are provided based on shared interests, correlated activities, and a number of other input signals.  GraphJet maintains a [realtime bipartite interaction graph](https://mathworld.wolfram.com/BipartiteGraph.html) that  keeps track of user–tweet interactions over the most recent n hours and reads from Kafka. Each individual GraphJet serever can ingest one million graph edges per second and compute 500 recommendations/second. 

They describe the reasons specifically for creating an in-memory DB in the GraphJet paper: 

> In terms of recommendation algorithms, we have found that random walks, particularly over bipartite graphs, work well for generating high-engagement recommendations. Although conceptually simple, random-walk algorithms define a large design space that supports customization for a wide range of application scenarios, for recommendations in different contexts (web, mobile, email digests, etc.) as well as entirely unrelated applications (e.g., social search). The output of our random-walk algorithms can serve as input to machine-learned models that further increase the quality of recommendations, but in many cases, the output is sufficiently relevant for direct user consumption.

> In terms of production infrastructure for generating graph recommendations, the deployed systems at Twitter have always gone "against the grain" of conventional wisdom. When many in the community were focused on building distributed graph stores, we built a solution (circa 2010) based on retaining the entire graph in memory on a single machine (i.e., no partitioning). This unorthodox design decision enabled Twitter to rapidly develop and deploy a missing feature in the service (see Section 2.1). Later, when there was much activity in the space of graph processing frameworks rushing to replace MapReduce, we abandoned the in-memory system and reimplemented our algorithms in Hadoop MapReduce (circa 2012). Once again, this might seem like another strange design decision (see Section 2.2). Most recently, we have supplemented Hadoop-based recommendations with custom infrastructure, first with a system called MagicRecs (see Section 2.3) and culminating in GraphJet, the focus of this paper.

The precursor to GraphJet was WTF, Who to Follow, which focused [only on recommending users to other users.](https://web.stanford.edu/~rezab/papers/wtf_overview.pdf), using [Cassovary, an in-memory graph processing engine](https://github.com/twitter/cassovary) built specifically for WTF, also built on the JVM. 

<img width="447" alt="Screenshot 2023-04-02 at 12 22 15 PM" src="https://user-images.githubusercontent.com/3837836/229365667-96855c3a-6238-4138-bdd7-faf396d8397e.png">

GraphJet implements two random walk algorithms:  
 + Circle of Trust (internal to Twitter) and 
 + SALSA (Stochastic Approach for Link-Structure Analysis). 

<img width="423" alt="Screenshot 2023-04-02 at 12 38 16 PM" src="https://user-images.githubusercontent.com/3837836/229366503-3f860693-6dfd-4755-90c4-e67c85964700.png">
GraphJet Architecture

+ A large portion of the traffic to GraphJet comes from
clients who request content recommendations for a partic-
ular user.

GraphJet includes [CLICK, FAVORITE, RETWEET, REPLY, AND TWEET as input node types](https://github.com/twitter/the-algorithm/blob/ec83d01dcaebf369444d75ed04b3625a0a645eb9/src/scala/com/twitter/recos/graph_common/NodeInfoHandler.scala#L22) and keeps track of left (input) and right(output) nodes. 

### SimClusters
--- 

[SimClusters](https://github.com/twitter/the-algorithm/tree/main/simclusters-ann) is another recommended tweet candidate generation source that, given an embedding (or a model-learned vector representation of an entity, in this case a tweet, event, or topic), will return a group of candidate tweets (or events, or topics) that are similar to the input content via [approximate nearest neighbors](https://en.wikipedia.org/wiki/Nearest_neighbor_search#Approximate_nearest_neighbor) lookup using [approximate cosine similarity as a distance metric](https://github.com/twitter/the-algorithm/blob/main/simclusters-ann/README.md#simclusters-approximate-cosine-similarity-core-algorithm). 

SimClusters are [built using this algorithm](https://github.com/twitter/the-algorithm/blob/main/src/scala/com/twitter/simclusters_v2/README.md) and served using the [ANN service](https://github.com/twitter/the-algorithm/tree/main/simclusters-ann).



## SimClusters Embeddings Algorithm
[See Paper here](https://dl.acm.org/doi/10.1145/3394486.3403370)

There is a multitude of content that can be recommended on Twitter: Tweets, user recommendations, events, hashtags, and Who to Follow, the service discussed in the GraphJet and [Who to Follow papers](https://web.stanford.edu/~rezab/papers/wtf_overview.pdf). 

Each of these recommendation algorithms need to be relearned frequently since content on Twitter moves quickly, and they need to be presented in a variety of places: not only in the feed, but also via email or notifications, or the "Trends and Events" section. 

However, the speed of change for different features is different: for example user recommendations change much more slowly than topic and tweet recommendations. Tweet embeddings are critical for tweet recommendation tasks. We can calculate [tweet similarity and recommend similar tweets to users based on their tweet engagement history.](https://github.com/twitter/the-algorithm/blob/main/src/scala/com/twitter/simclusters_v2/summingbird/README.md)

<img width="463" alt="Screenshot 2023-04-04 at 6 36 16 AM" src="https://user-images.githubusercontent.com/3837836/229766337-7b2a06cf-f38d-4c5d-b9b3-6f6221bc8d4f.png">

Previous iterations of homogenous recommendations for specific domains included WTF and GraphJet, but each hard their limitations. For example, GraphJet can generate a bipartite graph for recommendations at real time, but does not generalize to new domain areas. 

We can generalize all of these representations and learn them all from a single service, which was the idea behind SimClusters. 

In SimClusters, we don't use traditional recsys methods (i.e. matrix factorization) because it's computationally expensive in this case. Instead, we base it on ANN and construct a user-user cluster graph based on community structures (i.e. K-Pop or machine learning) where the central figures in the community are "influencers", and content is recommended on a  per-community level. 

The algorithm for community detection is based on [Metropolis-Hastings](https://en.wikipedia.org/wiki/Metropolis%E2%80%93Hastings_algorithm), computed offline on MapReduce jobs in Hadoop. 

It can compute clusters for 1 bil users (note that Twitter has [450 million MAU -monthly active users](https://www.demandsage.com/twitter-statistics/), which means that the clusters are computed on all potential users? ) and 100k dimensions, with each dimension representing a specific community, which allows it to represent [long-tail content fairly well.](https://arxiv.org/abs/2110.04596) 

## Simclusters Engineering Implementation
First, we ingest input data that includes tweets, topics, etc. It's unclear where this data comes from. 

**Stage 1:** Looking at the user-user graph and creating a list of communities that users belong to - "User Interest Representations" This is run as an MR job in Hadoop. 

**Stage 2:**  Calculates the representations for a given target, given a user-user bipartite graph, running in parallel. 

<img width="487" alt="Screenshot 2023-04-04 at 6 44 52 AM" src="https://user-images.githubusercontent.com/3837836/229768416-a3098145-d483-4eec-b574-70eaaf728f90.png">

Both of these stages can be run independently. 

Embeddings are generated and can be extended to work in batch-distributed, batch-multicore, or streaming-distributed modes. 

They are then used downstream and blended with sources in the [CR mixer.](https://github.com/twitter/the-algorithm/tree/main/cr-mixer) 

There is also a realtime component, [SimClusters ANN](https://github.com/twitter/the-algorithm/blob/main/simclusters-ann/README.md), which can return similar content based on the SimClusters output embedding. A [Heron](https://blog.twitter.com/engineering/en_us/topics/open-source/2018/heron-donated-to-apache-software-foundation) job builds the mapping between SimClusters and Tweets. The job saves top 400 Tweets for a SimClusters and top 100 SimClusters for a Tweet.

### Filters

If a user [opted out of](https://github.com/twitter/the-algorithm/blob/138bb519975407d4ea0dc1478d897d451ef05dab/src/scala/com/twitter/simclusters_v2/scalding/optout/SimClustersOptOutUtil.scala#L22) an interest group or category, they're ignored from the SimClusters index. 

### Hypothetical Architecture Diagram
<img width="487" alt="Simclusters" src="https://user-images.githubusercontent.com/3837836/230660246-6ce676c5-204d-47fa-9909-013565bec142.png">


### TwHIN
--- 

### RealGraph
--- 


### TweepCred
--- 

### Earlybird
+ The largest candidate generator is [Earlybird](https://blog.twitter.com/engineering/en_us/a/2011/the-engineering-behind-twitter-s-new-search-experience), a Lucene based real-time retrieval engine. There is an [Earlybird paper.](http://notes.stephenholiday.com/Earlybird.pdf) 

## Mixers

Before candidate tweets are sent to be ranked in the light and heavy rankers before being presented to the user, they are combined from their various candidate generation sources within several entites, one being the [CR Mixer,](https://github.com/twitter/the-algorithm/blob/main/cr-mixer/README.md) which fetches out-of-network recommended candidate tweets.

### CR Mixer

## Rankers

Based on the Twitter engineering blog post, a total of 1500 candidates are retrieved. However, only some of them will be served to your Twitter feed.

Twitter would want to show the tweets that you are most likely to positively engage with. Therefore Twitter will predict probabilities of whether you will engage with the tweet, and use these probabilities to score the tweets.

To reduce computation cost, tweets are first ranked with a light ranker (which is just a logistic regression) and then a heavy ranking (a neural network model).

### Light Ranker

This is their [documentation](https://github.com/twitter/the-algorithm/blob/main/src/python/twitter/deepbird/projects/timelines/scripts/models/earlybird/README.md)

- “The Earlybird light ranker is a which predicts the likelihood that the user will engage with a tweet. It is intended to be a simplified version of the heavy ranker which can run on a greater amount of tweets.”
- “The current model was last trained several years ago, and uses some very strange features. We are working on training a new model, and eventually rebuilding this part of the stack entirely.”

Twitter has separate models for ranking in-network and out-network tweets, with different features
- [In-network model](https://github.com/twitter/the-algorithm/blob/main/src/python/twitter/deepbird/projects/timelines/configs/recap_earlybird/feature_config.py)
- [Out-of-network model](https://github.com/twitter/the-algorithm/blob/main/src/python/twitter/deepbird/projects/timelines/configs/rectweet_earlybird/feature_config.py)

The model for the Light Ranker TensorFlow model is trained using [TWML](https://github.com/twitter/the-algorithm/blob/main/twml/README.md) which is said to be deprecated, but the code is in [deepbird](https://github.com/twitter/the-algorithm/blob/ec83d01dcaebf369444d75ed04b3625a0a645eb9/src/python/twitter/deepbird/projects/timelines/scripts/models/earlybird/README.md) project.

The Earlybird Light Ranker has some [feature weights](https://github.com/twitter/the-algorithm/blob/ec83d01dcaebf369444d75ed04b3625a0a645eb9/src/python/twitter/deepbird/projects/timelines/scripts/models/earlybird/example_weights.py#L6) but as suggested in the code, they are read in as run time parameters and these are most likely different in practice.

### Heavy Ranker

+ **Recap** The "Heavy Ranker" is a [parallel masknet](https://arxiv.org/abs/2102.07619). Majority of the code for this is in the [ML repo](https://github.com/twitter/the-algorithm-ml/blob/main/projects/home/recap/README.md). The [ranker itself](https://github.com/twitter/the-algorithm-ml/blob/main/projects/home/recap/README.md) is run after the candidate generators. 

It's important to note that [there are no content-based embeddings](https://twitter.com/YingXiao/status/1643398843562856450) inside the main ranking algorithm.

[Input features](https://github.com/twitter/the-algorithm-ml/blob/main/projects/home/recap/FEATURES.md). All the specific features within the input feature list are based slightly on content signals and mostly social signals, such as "aggregate counts of user interaction with other engagers of tweets that the user interacts with", and based heavily on "likes" and "replies" as input actions, but at an aggregate level. The social and embeddings-based features in the dataset are not used and weighted as much. 

All of these are combined and weighted into a score. Hyperparameters for the model [and weighting are here.](https://github.com/twitter/the-algorithm-ml/blob/78c3235eee5b4e111ccacb7d48e80eca019e480c/projects/home/recap/config/local_prod.yaml#L1)

For more details on the model, see the [Architecture overview](masknet.md).


### Scoring Plan

After the model predicts the probability of the actions, weights are assigned to the probability. The tweet with the highest score is likely to appear at the top of your feed.

These are the actions predicted, and [their corresponding weights](https://raw.githubusercontent.com/twitter/the-algorithm-ml/main/projects/home/recap/README.md)

| feature                                                                          | weight |
|----------------------------------------------------------------------------------|--------|
|probability the user will favorite the Tweet |(0.5)|
|probability the user will click into the conversation of this tweet and reply or like a Tweet |(11*)|
|probability the user will click into the conversation of this Tweet and stay there for at least 2 minutes |(11*)|
|probability the user will react negatively (requesting "show less often" on the Tweet or author, block or mute the Tweet author) |(-74)|
|probability the user opens the Tweet author profile and Likes or replies to a Tweet |(12)|
|probability the user replies to the Tweet |(27)|
|probability the user replies to the Tweet and this reply is engaged by the Tweet author |(75)|
|probability the user will click Report Tweet |(-369)|
|probability the user will ReTweet the Tweet |(1)|
|probability (for a video Tweet) that the user will watch at least half of the video |(0.005)|

The score of the tweet is equal to

```
P(favorite) * 0.5 + max( P(click and reply), P(click and stay two minutes) ) * 11 + P(hide or block or mute) * -74 + ... etc
```

The tweet with the highest score is likely to appear at the top of your feed. (There is still a part on boost where multipliers will be applied to the score). However, filtering is applied afterwards, and this could change what tweets you actually see.

There are some interpretations we can make from the scoring plan

- They combine the negative feedback actions (hide/mute/block) even though they have different produce consequences. By combining the predictions I think they hope to generalize the signal. However, the report prediction is by itself and has a much larger negative weight.
- There is very limited implicit action in the scoring plan. This is unlike short video recommendation systems like TikTok where the system learns from how long you stay on the video. The weight for the video completion prediction is insignificant.
- The only implicit action being predicted is when you click into the conversation of this Tweet and stay there for at least 2 minutes. 2 minutes is quite a large number. This can be viewed as a defense against comment bait, where the author entices you to click on the comments but leave you disappointed. If you exit the comment section soon after clicking, it is not considered a positive signal to engagement.
- The scoring plan encourages participation in the conversation. The weight for the probability of you replying is high. The weight for the probability of the author replying to your reply is even higher. We can view this as Twitter's intention to be the "town square" of the Internet. However, this signal does not differentiate whether the conversation is friendly or otherwise (unless you also hide/mute/block/report).
- We should also note that the score of Blue Verified authors will be given a [multiplier of 4 or 2](https://github.com/twitter/the-algorithm/blob/d1cab28a1044a147a107ae067890850041956777/home-mixer/server/src/main/scala/com/twitter/home_mixer/param/HomeGlobalParams.scala#L89,L103), which overrides many of the weights in the scoring plan.

The release does not describe how the weights are chosen. We expect the weights to be tuned with A/B testing. We are also curious about what Twitter measures and optimizes when they tune the weights.

## Filters

Usually, filtering happens before ranking to avoid the need to rank candidates that will be filtered later. However, on Twitter, the blog implies that filtering happens after ranking.

+ [visibility-filters](https://github.com/twitter/the-algorithm/blob/main/visibilitylib/README.md)
  + (From the blog) "Filter out Tweets based on their content and your preferences. For instance, remove Tweets from accounts you block or mute."
  + “Visibility Filtering library is currently being reviewed and rebuilt, and part of the code has been removed and is not ready to be shared yet. The remaining part of the code needs further review and will be shared once it’s ready. Also code comments have been sanitized.”

+ Remove [out-of-network competitor site URLs](https://github.com/twitter/the-algorithm/blob/main/home-mixer/server/src/main/scala/com/twitter/home_mixer/functional_component/filter/OutOfNetworkCompetitorURLFilter.scala) from potential offered candidate Tweets


## Ordering

There are some reasons why we might not want to order the tweets strictly by the scoring plan. The scoring plan scores tweets independent of other Tweets. However, we might want to consider other tweets when presenting the tweets on the feed, for example, avoid showing tweets from the same author consecutively or maintain some other form of diversity in the tweets.

These are the heuristics mentioned in the [blog](https://blog.twitter.com/engineering/en_us/topics/open-source/2023/twitter-recommendation-algorithm)

- **Author Diversity**: Avoid too many consecutive Tweets from a single author.
  - See [code](https://github.com/twitter/the-algorithm/blob/main/home-mixer/server/src/main/scala/com/twitter/home_mixer/product/scored_tweets/scorer/DiversityDiscountProvider.scala)
  - `score * ((1 - 0.25) * Math.pow(0.5, position) + 0.25)`
  - If you have seen the author is the same feed refresh, the score of the tweet from the author havled (but with a floor)

- **Content Balance**: Ensure we are delivering a fair balance of In-Network and Out-of-Network Tweets.
  - (Contributions needed)

- **Feedback-based Fatigue**: Lower the score of certain Tweets if the viewer has provided negative feedback around it.
  - See [code](https://github.com/twitter/the-algorithm/blob/main/home-mixer/server/src/main/scala/com/twitter/home_mixer/functional_component/scorer/FeedbackFatigueScorer.scala)
  - The multiplier will be less than one if you
    - Provided negative feedback on the author of the tweet
    - Provided negative feedback to the users who like the tweet
    - Provided negative feedback on users who follow the author of the tweet (?)
    - Provided negative feedback on users who retweeted the tweet
  - Recent negative feedback will have a greater weight
    - If the negative feedback is provided more than 14 + 140 days ago, the negative feedback will not be considered.
    - If the negative feedback was provided less than 14 days ago, the tweet will be filtered. See [code](https://github.com/twitter/the-algorithm/blob/main/home-mixer/server/src/main/scala/com/twitter/home_mixer/functional_component/filter/FeedbackFatigueFilter.scala)

- **Social Proof**: Exclude Out-of-Network Tweets without a second degree connection to the Tweet as a quality safeguard. In other words, ensure someone you follow engaged with the Tweet or follows the Tweet’s author.
  - What is described above is a filter, not a discount. However, we can find the discount, see [code](https://github.com/twitter/the-algorithm/blob/main/home-mixer/server/src/main/scala/com/twitter/home_mixer/functional_component/scorer/OONTweetScalingScorer.scala)
  - `ScaleFactor = 0.75` is applied to out-of-network tweets (exactly second degree connection?), in-network retweets of out-of-network tweets should not have this multiplier applied
  - We might have a filter that removes all content with more than two degrees of connection.

- **Twitter Blue boost**: (Not listed in blog)
  - See [code](https://github.com/twitter/the-algorithm/blob/main/home-mixer/server/src/main/scala/com/twitter/home_mixer/functional_component/scorer/VerifiedAuthorScalingScorer.scala) and [default parameters](https://github.com/twitter/the-algorithm/blob/main/home-mixer/server/src/main/scala/com/twitter/home_mixer/param/HomeGlobalParams.scala#L89)
    - If the author of the candidate tweet is a Blue Verified and in the network of the user (i.e. user follows author?), the score of the tweet is multiplied by 4
    - If the author of the candidate tweet is a Blue Verified and out of the network of the user (i.e. does not follow author an within two degrees of connection), the score of the tweet from is multiplied by 2.
  - This means that Blue Verified authors that the user does not follow is given a greater boost than the authors the user explictly follows.
  - Note that Twitter Blue is launched shortly after Elon Musk's takeover.




## Business Terms and Logic

These are Twitter specific terms and names that keep coming up across different code bases and blog posts.

+ Twepoch - A "magic number" `1288834974657L`, which is a timestamp for `2010-11-04T01:42:54Z` the date that Twitter introduced the Snowflake ID system, used as Twitter's own ["Unix Epoch"](https://en.wikipedia.org/wiki/Epoch_(computing))
+ Snowflake - Twitter's system for [assigning unique IDs](https://blog.twitter.com/engineering/en_us/a/2010/announcing-snowflake) to tweets, users, lists, DMs, media etc. 
+ WTF - [Who to follow](https://web.stanford.edu/~rezab/papers/wtf_overview.pdf)
+ DDG - Duck Duck Goose, Twitter's [A/B Testing Platform](https://blog.twitter.com/engineering/en_us/a/2015/twitter-experimentation-technical-overview).
+ Earlybird - Twitter's Lucene based real-time search index. [Notes](https://stephenholiday.com/notes/earlybird/) and a [blog post here](https://blog.twitter.com/engineering/en_us/a/2011/the-engineering-behind-twitter-s-new-search-experience).
+ "Unregretted user minutes" - the metric Twitter publicly states is the thing they are optimizing for. It is unknown how exactly they measure this.

## Bias and Manipulation

Cases of potential bias, manipulation, favouritism, hacks, etc. The focus on this repository is on the concrete, techincal aspects of the code, not speculating on anything twitter may or may not have done. That exercise is left to the reader, however, there are some technical aspects that should still be described about these popular accusations, this is a section for those. Unfortunately, much of the configuration that would contain specific instances of interventions is not in the code.

### Deboosting Rival Sites

It was long speculated youtube links get massively deboosted, and Spaces links massively boost Tweets in recommendations. There are no specific references to this in the code. However, there are filters that could be configured for this, referencing [OutOfNetworkCompetitorURLFilter](https://github.com/twitter/the-algorithm/blob/ec83d01dcaebf369444d75ed04b3625a0a645eb9/home-mixer/server/src/main/scala/com/twitter/home_mixer/product/scored_tweets/ScoredTweetsRecommendationPipelineConfig.scala#L231) for example.

### Elon Musk feature

The Elon Musk / Democrat / Republican Code: [Now Removed](https://github.com/twitter/the-algorithm/commit/ec83d01dcaebf369444d75ed04b3625a0a645eb9). One of the first widely shared cases, falsely assuming this is something that directly affects Recommendations when it was actually for internal A/B testing, to monitor for effects (DDG is Duck Duck Goose, the A/B Testing Platfom). It was also mentioned in [the space](https://twitter.com/elonmusk/status/1641880448061120513) and denied there. However, a former twitter employee also offered an [alternative explanation](https://twitter.com/igb/status/1641910616939167745) (A/B Testing measures behavior, so one way or another Twitter is tuning your TL, indirectly).

### Ukraine
There are two mentions related to Ukraine in the Twiter Algo repo. Whereas [one of them](https://github.com/twitter/the-algorithm/blob/7f90d0ca342b928b479b512ec51ac2c3821f5922/visibilitylib/src/main/scala/com/twitter/visibility/rules/PublicInterestRules.scala#L54) is a flag for Ukraine-related misinformation used for moderation, or warning labels, there is another _safety label_ for Twitter Spaces called _[UkraineCrisisTopic](https://github.com/twitter/the-algorithm/blob/7f90d0ca342b928b479b512ec51ac2c3821f5922/visibilitylib/src/main/scala/com/twitter/visibility/models/SpaceSafetyLabelType.scala#L39)_. Here are some facts about these labels and their function:
  * Each safety label "[describes a particular policy violation, and usually leads to reduced visibility of the labeled entity in product surfaces](https://github.com/twitter/the-algorithm/blob/main/visibilitylib/README.md)"
  * `SafetyLabel` results in tweet interstitial or notice, are publicly [documented here previously](https://help.twitter.com/en/resources/addressing-misleading-info) and specifically for [Armed Conflicts here](https://help.twitter.com/en/rules-and-policies/crisis-misinformation).
  * All [other Twitter Spaces safety labels](https://github.com/twitter/the-algorithm/blob/7f90d0ca342b928b479b512ec51ac2c3821f5922/visibilitylib/src/main/scala/com/twitter/visibility/models/SpaceSafetyLabelType.scala#L26) are related to misinformation, NSFW, toxic or harmful content, DMCA takedowns, etc.

## Changes

+ 2 hours after it was released, [Twitter removed](https://github.com/twitter/the-algorithm/compare/ef4c5eb65e6e04fac4f0e1fa8bbeff56b75c1f98...ec83d01dcaebf369444d75ed04b3625a0a645eb9) feature flags that specifically higlighted Elon's account

## Discussions about The Algorithm Elsewhere

+ [What can we learn from `The Algorithm,' Twitter's partial open-sourcing of it's feed-ranking recommendation system?](https://solomonmg.github.io/post/twitter-the-algorithm/)
+ A CVE (Common Vulnerabilities and Exposures) [CVE-2023-29218](https://nvd.nist.gov/vuln/detail/CVE-2023-29218) was drafted for a potential denial of service attack using coordinated mass blocking, muting and unfollowing, citing the code release (but the attack itself was described, discussed, and widely speculated on previously).

## Resources for Learning More about Recsys

+ @karlhigley's [blog](https://practicalrecs.com/the-rest-of-the-owl.html) and [thread of threads](https://twitter.com/karlhigley/status/1138122648934916097) are very accessible things about Recommender Systems in practice.

+ A good Recommender Systems entry point is the Google Machine Learning for [Recommender Systems course](https://developers.google.com/machine-learning/recommendation), it also has a good [glossary](https://developers.google.com/machine-learning/glossary/recsystems) of terms.

+ The biggest academic recsys community is [ACM Recsys](https://recsys.acm.org/) and state-of-the-art recommender systems research is usually openly available in the [proceedings](https://dl.acm.org/conference/recsys). A lot of the presentations are on [youtube](https://www.youtube.com/@acmrecsys/videos).

+ Admittedly out of date, but still useful, is the [RecSys Wiki](https://www.recsyswiki.com/wiki/Main_Page).

+ The latest edition of the [Recommender Systems Handbook](https://link.springer.com/book/10.1007/978-1-0716-2197-4) is also a good book that covers the field well.

+ I find it very helpful to break down recommendation systems into four stages - [retrieval, filtering, scoring, and ordering](https://medium.com/nvidia-merlin/recommender-systems-not-just-recommender-models-485c161c755e).

- Although a [systems design interview guide](http://patrickhalina.com/posts/ml-systems-design-interview-guide/), it introduces the most important design consideration for a recommendation system.
