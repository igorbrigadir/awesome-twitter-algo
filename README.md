# Awesome Twitter Algo :bird:

An annotated look through the release of the Twitter algorithm, through the context of engineering and recsys, with notes from repo creators on significance of specific parts of the code. 

Curated by [Igor Brigadir](https://github.com/igorbrigadir) and [Vicki Boykis](https://github.com/veekaybee)

# Source Data 
[Twitter Algo Repo](https://github.com/twitter/the-algorithm) || [Twitter ML Algo Repo](https://github.com/twitter/the-algorithm-ml) || [Blog Post](https://blog.twitter.com/engineering/en_us/topics/open-source/2023/twitter-recommendation-algorithm)

# Architecture

# Input Data

+ The system starts with [500 million tweets posted on a daily basis](https://blog.twitter.com/engineering/en_us/topics/open-source/2023/twitter-recommendation-algorithm). It filters to showing you one of 1500 possible tweet generated candidates. 


# Candidate Generators


# Rankers



# Filters


# Business Terms and Logic

+ [Twepoch](https://github.com/twitter/the-algorithm/blob/ec83d01dcaebf369444d75ed04b3625a0a645eb9/src/java/com/twitter/search/earlybird_root/filters/ResultTierCountFilter.java#L106), `2010-11-04T01:42:54Z` is the release date of Twitter
+ WTF - [Who to follow](https://web.stanford.edu/~rezab/papers/wtf_overview.pdf)

# Changes

+ 2 hours after it was released, [Twitter removed](https://github.com/twitter/the-algorithm/commit/ec83d01dcaebf369444d75ed04b3625a0a645eb9) feature flags that specifically higlighted Elon's account
