A topic model for the Twenty Newsgroups data
============================================
[LDAvis](https://github.com/cpsievert/LDAvis/) comes prepackaged with some data sets to help quickly demonstrate how to use it. This document visualizes a topic model fit to the 'Twenty Newsgroups' data created with **LDAvis** and **knitr** ([see here](https://github.com/cpsievert/LDAvis/blob/master/inst/examples/newsgroup/newsgroup.Rmd) for source code). 

First, we downloaded the data from the [home page for the Twenty Newsgroups data](http://qwone.com/~jason/20Newsgroups/). Specifically, we used the '20news-bydate' version of the data and fit our topic model to the 'training' portion of the data. The raw training data consists of $D = 11,269$ documents, a vocabulary of $W = 53,975$ terms, and $N = 2,765,300$ total tokens in the corpus. Each document is a message posted to one of twenty selected Usenet newsgroups during a time span roughly between June, 1992 and May, 1993. It appears that the documents were tokenized by splitting on punctuation and whitespace. We remove all occurrences of the 174 stop words contained in the "English" stop words list in the R package **tm**. We also removed all occurrences of terms that occurred less than a total of ten times. This left $W = 15,954$ terms in the vocabulary and a total of $N = 1,511,137$ tokens in the corpus. One document was removed because it contained only stop words and rare words.

We fit a $K=50$-topic model to the corpus (allowing for topics other than the 20 standard newsgroups topics to be discovered) by running the collapsed Gibbs sampler for 10,000 iterations using symmetric priors for the document-topic distributions ($\alpha = 0.02$) and the topic-term distributions ($\beta = 0.02$). We used MALLET to fit the model. We computed estimates of the document-topic distributions (stored in a $D \times K$ matrix denoted $\theta$) and the topic-term distributions (stored in a $K \times W$ matrix denoted $\phi$) by cross-tabulating the latent topic assignments from the last iteration of the Gibbs sampler with the document IDs and the term IDs, and then adding pseudocounts to account for the priors. A better estimate might average over multiple MCMC iterations of latent topic assignments (assuming the MCMC has settled into a local mode of the posterior and there is no label-switching going on), but we don't worry about that for now.

To visualize the fitted model using `LDAvis`, we load the data object `TwentyNewsgroups`, which is a list containing five elements.


```r
library(LDAvis)
data("TwentyNewsgroups", package = "LDAvis")
str(TwentyNewsgroups)
```

```
## List of 5
##  $ phi           : num [1:50, 1:15954] 5.78e-07 1.26e-04 2.77e-06 2.98e-04 4.55e-07 ...
##  $ theta         : num [1:11268, 1:50] 2.28e-05 2.08e-04 6.06e-04 5.88e-04 1.02e-04 ...
##  $ doc.length    : int [1:11268] 878 95 32 33 196 23 44 83 38 179 ...
##  $ vocab         : chr [1:15954] "archive" "name" "atheism" "resources" ...
##  $ term.frequency: int [1:15954] 317 1364 300 226 327 1832 125 108 1002 208 ...
```

The first two elements are $\phi$ and $\theta$. Both of these are matrices whose rows must sum to one, since their rows contain probability distributions over terms and topics, respectively.

The third element of the list is `doc.length`, which is an integer vector of length $D = 11,268$ containing the number of tokens in each document. For this data the median document length is 81 tokens, with a range of 1 to 6409.

The fourth element of the list is `vocab`, which is a character vector containing the terms in the vocabulary, in the same order as the columns of $\phi$.

The fifth element of the list is `term.frequency`, which is an integer vector containing the frequencies of the terms in the vocabulary. The median term frequency is 27 with a range of 10 to 12,289 ('edu' is the most frequent term, because the data contain email addresses and tokenization was performed by splitting on punctuation).

At this point, we call the R function `createJSON()` to create a JSON object that will feed the web-based visualization. The `createJSON()` function performs several operations:

- It computes topic frequencies, inter-topic distances, and a projection of the topics onto a two-dimensional space using multidimensional scaling.

- It computes the $R$ most relevant terms for each topic for a grid of values of $\lambda$ (determined by the argument `lambda.step`, set to 0.01 by default), where the relevance of a term to a topic is defined as $\lambda \times p(term \mid topic) + (1 - \lambda) \times p(term \mid topic)/p(term)$, for $0 \leq \lambda \leq 1$.

The idea is to help users interpret topics by allowing them to interactively re-rank the most relevant terms for each topic by changing the value of $\lambda$ via a slider, where large values of $\lambda$ highly rank frequent words within a topic, and low values of $\lambda$ highly rank exclusive words within a topic. The topic plot on the left side of **LDAvis** allows users to browse groups of similar topics (positioned near each other in the 2-d plot) or simply progress through the topics in order (they are, by default, ordered in decreasing order of frequency). For more on relevance, see our paper about **LDAvis** [here](http://nlp.stanford.edu/events/illvi2014/papers/sievert-illvi2014.pdf).


```r
json <- with(TwentyNewsgroups, 
             createJSON(phi = phi, theta = theta, vocab = vocab,
                doc.length = doc.length, term.frequency = term.frequency))
```

Now, the `serVis` function can take `json` and serve the result in a variety of ways. Here we write `json` to a file within the 'vis' directory (along with other HTML and JavaScript required to render the page). You can see the result [here](http://cpsievert.github.io/LDAvis/newsgroup/vis).


```r
serVis(json, out.dir = "vis", open.browser = FALSE)
```

