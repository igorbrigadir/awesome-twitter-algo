# Twitter Model Architecture Summary

## Summary

Because of the high number of categorical features and important interaction effects, Ads/RecSys models often use models which specifically incorporate cross features (the interaction of two features). We review a couple different approaches to this problem and explain how MaskNet, the model used by the Twitter heavy ranker, works.

## Background

Let's suppose you have a friend @username on Twitter and you only retweet their tweets about cats. You follow them, but you don't want to see their tweets about dogs (on Twitter these days people only like cats).

To understand the probability that you will retweet their tweet, the model needs to understand the interaction between a feature like "tweet is about cats" and "author is @username". We refer to this as a feature cross and it is a common problem in Ads/RecSys models.

So, how do we model this interaction?

## Base Models: Polynomial Models

The classical approach to modeling feature crosses is to use a linear polynomial model. This is a model which is a linear combination of the features and their powers. For example, if we have two features x and y and we want to model the second-order interaction between x and y, we would use a model like:

$$\hat{y} = \beta_0 + \beta_1 x + \beta_2 y + \beta_4 xy +...$$

To make this a little more concrete, let's assume we have a model with only two features: a categorical feature for tweet topic and a categorical feature for user id and let's one hot encode each of the features into a group of features.

Now consider the second order features which we can form by multiplying any given one-hot feature with another one-hot feature. In this setup, we only have 3 non-zero second-order features:

- topic = cats AND author = @username (the cross feature we wanted)
- topic = cats AND topic = cats (which reduces to topic = cats, already in the model)
- author = @username AND author = @username (collinear as well)

All other second-order features are zero in this one-hot encoding, and so for this model we can learn the strength of the interaction of tweets about cats for @username for this user session (and can generalize this to include higher-order interactions like viewer = you AND topic = cats AND author = @username).

However, the problem with this approach is that it is computationally expensive to train and it is difficult to scale to large numbers of features. Unfortunately, to consider the second order interaction between all features, we would need to train a model with $O(n^2)$ features and so on.

Feature selection or hand-picking features is a common approach to this problem. However, this is not a scalable solution for features with high cardinality (like authors) and it is difficult to know which features to select.

Let's write the general form of our linear interaction model (order 2):

$$\hat{y} = \beta_0 + \sum_{i=1}^n \beta_i x_i + \sum_{i=1}^n \sum_{j=i+1}^n w_{ij} x_i x_j$$

To make the next model more intuitive, let's rewrite this into matrix form:

$$\hat{y} = \beta_0 + \mathbf{X} \mathbf{\beta} + \mathbf{X} \mathbf{W} \mathbf{X}^T$$

where $\mathbf{X}$ is a matrix of features, $\mathbf{\beta}$ is a vector of coefficients, and $\mathbf{W}$ is a matrix of weights.

In particular, $\mathbf{W}$ is a symmetric matrix with $w_{ij} = w_{ji}$ where each element is the weight for the interaction between feature $i$ and feature $j$. There are $n(n-1)/2$ unique elements in this matrix.

## Factorization Machines

In a Factorization Machine (FM), we factorize the weights matrix $\mathbf{W}$ into a matrix decomposition $\mathbf{V} \mathbf{V}^T$.

$$\hat{y} = \beta_0 + \mathbf{x} \mathbf{\beta} + \mathbf{x} \mathbf{V} \mathbf{V}^T \mathbf{x}^T$$

where $\mathbf{V}$ is a matrix of latent factors, or to use the terminology from NLP, embeddings. (One of the key insights of NLP is that training Word2Vec embeddings is equivalent to matrix factorization, see the references).

Writing this in another form, we can see that the FM is a linear model with a feature cross term:

$$\hat{y} = \beta_0 + \sum_{i=1}^n \beta_i x_i + \sum_{i=1}^n \sum_{j=i+1}^n \langle \mathbf{v}_i, \mathbf{v}_j\rangle x_i x_j$$

where $\mathbf{v}_i$ is the embedding for feature $i$. Comparing with above, we can see that the dot product of the embeddings is the weight for the interaction between feature $i$ and feature $j$.

Factorization Machines in particular have a neat trick called the sum-of-squares trick where we can reduce the computation to be linear in the number of features.

## Wide & Deep Models

Another generalization of our linear polynomial model is known as the Wide & Deep model. In this model, we replace the linear part of the model with a neural network while leaving select higher-order feature crosses in the "wide" part of the model.

We can think of this as a feature partition between deep features $x_d$ and wide features $x_{ij}$:

$$\hat{y} = f(\beta_0 + \sum_{d=1}^n \beta_i x_d) + \sum_{ij=1}^m w_{ij} x_{ij}$$

where $f$ is a neural network (probably a multi-layer perceptron).

The idea here is that the deep part of the network can learn a complex non-linear transformation of the dense features while the wide part of the network keeps important cross features directly in the last layer.

## Deep Factorization Machines (DLRM variant)

The Deep Factorization Machine (DLRM variant) is a generalization of a Factorization Machine that also incorporates a feature partition between dense and sparse features. We can write it as:

$$\hat{y} = g(f(\beta_0 + \sum_{d=1}^n \beta_i x_d) + \sum_{i=1}^n \sum_{j=i+1}^n \mathbf{v}_i^T \mathbf{v}_j x_{ij})$$

where $f$ and $g$ are a neural network (probably a multi-layer perceptron).

Like the Wide & Deep model, we learn a transformation of the dense features and then add the feature crosses. However, unlike the Wide & Deep model, we use the embeddings to generate the feature crosses.

We then pass the concatenation of the dense projection and the feature crosses to a final neural network $g$ to generate the prediction.

## Deep Cross Net (v2)

A similar riff on this idea is the Deep Cross Net (v2) model. Unlike the DeepFM or Wide & Deep models, the Deep Cross Net (v2) model does not have a feature partition between dense and sparse features.

Instead, we generate feature crosses for all the features using a matrix equation very similar to our linear polynomial model (where the circle is element-wise multiplication):

$$\mathbf{x_0} \circ \mathbf{W} \mathbf{x_0}^T$$

The key insight here is that rather than taking the full product with $x_0$ leaving us with exactly 1 term (FM) or not taking it and having $n(n-1)/2$ unique terms (DeepFM), we can take an element-wise multiplication (aka the Hadamard product) with $x_0$ which leaves us with exactly $n$ terms.

For a given feature $x_i$, we can write its output simply as a cross between the feature and a linear combination of the other features:

$$x_i \sum_{j=1}^n w_{ij} x_j$$

This is a neat trick because we're able to generate the same number of (second-order) feature crosses as we have features. This means that we can feed the output of this equation back into our equation and generate the crosses between our second-order crosses and our original features (aka third-order crosses) and so on! This gives us the real equation:

$$x_{i+1} = \mathbf{x_0} \circ (\mathbf{W} \mathbf{x_i} + \mathbf{\beta}) + \mathbf{x_i}$$

This approach can be combined with a neural net output to generate the final prediction.

We can also use the Factorization Machine approach to factorize the weights matrix $\mathbf{W}$ into a matrix decomposition $\mathbf{U} \mathbf{V}^T$ (the original paper does not suggest a symmetric decomposition but this can be included).

## MaskNet

MaskNet can be thought of as a multi-head variant of the Deep Cross Net shown above. (Although the paper frames it in terms of an attention mechanism - which also uses element-wise multiplication - attention usually involves a softmax which is not the case here).

The MaskNet equation (for a single head) is:

$$\mathbf{x_0} \circ \mathbf{U} g(\mathbf{V}^T \mathbf{x_0}^T + \mathbf{\beta})$$

This introduces crosses via a matrix decomposition $\mathbf{U} \mathbf{V}^T$ where the intermediate expression is passed through a non-linear activation function $g$ (this is an extension explicitly contemplated in the DCN paper)

Finally, we also introduce LayerNorm's on the output of this, which is a common technique to stabilize training and improve generalization.

These parallel heads are then concatenated together and passed through a final neural network to generate the final prediction (or in this case, a collection of multi-task neural network heads produce a set of predictions).

To see the code for this, check out [this code](https://github.com/twitter/the-algorithm-ml/blob/main/projects/home/recap/model/mask_net.py#L47) in the Twitter repo (note that `net == mask_input`).

## References

- Factorization Machines: https://www.csie.ntu.edu.tw/~b97053/paper/Rendle2010FM.pdf
- DeepFM: https://arxiv.org/pdf/1703.04247.pdf
- Wide & Deep: https://arxiv.org/pdf/1606.07792.pdf
- DLRM: https://arxiv.org/pdf/1906.00091.pdf
- Deep Cross Net v2: https://arxiv.org/abs/2008.13535
- Google blog about Feature Crosses: https://www.tensorflow.org/recommenders/examples/dcn
- MaskNet: https://arxiv.org/abs/2102.07619
- Neural Word Embeddings as Implicit Matrix Factorization: https://papers.nips.cc/paper/2014/file/feab05aa91085b7a8012516bc3533958-Paper.pdf