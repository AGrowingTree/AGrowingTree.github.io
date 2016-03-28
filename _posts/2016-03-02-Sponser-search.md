---
author:
- Huayang Li
title: Sponser Advertising
...

INTRODUCTION
============

But for the SS, the web will be much more small. Once a user type the
queries and browse on the search engines, the advertiser will bid for
this queries and the winner’s Ad will be shown with the result. The
advertiser want the conversion, the network want the revenue, and the
user want the relevance. The cure problem to satisfy advertisers, users,
and the netwrork is to select the relevant textual ad.

there are 3 technique mentioned in the ppt, **EM(Exact Match)**, **AM**\
**(Advanced Match)**, and **PM(Phase Match)**. EM is using for the
frequently occured queries, we’ll talk in section 1. The most
complicated technique is AM. AM is born for the long tail of queries,
because more than 80% of queries occur one time. The challenge of AM is
queries re-writting, we’ll talk in section 2. For section 3, we’ll talk
the problem remain to be solved in the new term.

Implementation Approaches
-------------------------

1.  **The data base approach**(original Overture approach)

    -   Ads are records in a data base

    -   The bid phrase (BP) is an attribute

    -   On query q, for EM consider all ads with BP=q

2.  **The IR approach**(modern view)

    -   Ads are documents in an ad corpus

    -   The bid phrase is a meta-datum

    -   On query q run q against the ad corpus

        -   Have a suitable ranking function (more later)

        -   BP = q (exact match) has high weight

        -   No distinction between AM and EM

The two phases of ad selection
------------------------------

-   Ad Retrieval: Consider the whole ad corpus and select a set of most
    viable candidates (e.g. 100)

-   Ad Reordering: Re-score the candidates using a more elaborate
    scoring function to produce the final ordering

EXACT MATCH
===========

What’s EM?

-   queries = BP(bid phrase)

-   e.g. if the users query is “shoes”, then only the bid phrase “shoes”
    will match, the “sports shoes”, “shoe” or “shoees”… will fail
    to match.

-   this matching type will raise up the CTR but the volume will be low,
    in other words, it will reach less users.

ADVANCED MATCH
==============

Query Rewriting Basing On Query Logs
------------------------------------

### Definition of Query Pair

The data used comes from logs of user web accesses. This data contains
web searches annotated with user ID and timestamp. A *candidate
reformulation*is a pair of successive queries issued by a single user on
a single day. Candidate reformulations will also be referred to as query
pairs. $$\begin{aligned}
\text{candidate}&\text{Query}\text{Pairs}(user_{i},day_{i})=\{\langle q_{1},q_{2} \rangle:(q_{1}\ne q_{2})\land\\ &\exists t: query_{t}(user_{i}, q_{1}) \land query_{t+1}(user_{i}, q_{2})\}\end{aligned}$$

### Phrase Substitutions

The repeated searches for the same terms, as well as query pair
sequences repeated by the same user on the same day. We then aggregate
over users, so the data for a single day consists of all candidate
reformulations for all users for that day.

then use the phrases identified by high point-wise mutual information we
segment queries into phrase for example “(new york) (maps)” or “(britney
spears) (mp3s)”. where we set the threshold κ to be
8$$\frac{P(\alpha, \beta)}{P(\alpha),P(\beta)} > \kappa$$

### Identifying Significant Query Pairs and Phrase Pairs

In order to distinguish related query and phrase pairs from candidate
pairs that are unrelated, we use the pair indepen- dence hypothesis
likelihood ratio. This metric tests the hypothesis that the probability
of term q2 is the same whether term q1 has been seen or not, by
calculating the likelihood of the observed data under a binomial
distribution using probabilities derived using each hypothesis
$$\begin{aligned}
     H_{1}:P(q_{2}|q_{1})=&p=P(q_{2}|\neg q_{1})\\
H_{2}:P(q_{2}|q_{1})=p_{1}&\ne p_{2}=P(q_{2}|\neg q_{1})\\\end{aligned}$$
The likelihood score is $$\lambda = \frac{L(H_{1})}{L(H_{2})}$$ The test
statistic $-2 \log \lambda$ is asymptotically $\chi^{2}$ distributed.
Therefore we work with the log likelihood ratio score:
$$\text{LLR} = −2 \log \lambda = −2 \log \frac{L(H_{1})}{L(H_{2})}$$ A
high value for the likelihood ratio suggests that there is a strong
dependence between term $q_1$ and term $q_2$. We refer to query pairs
and phrase pairs above a threshold for the LLR score as
*substitutables*.

### Generating Candidates

We seek to generate statistically significant related queries for
arbitrary input queries. For frequent queries, we may have many
suggestions. But for less frequent queries, maybe there is no
statistically significant related queries. Because of the long tail, the
less frequent queries has a large fraction of the whole queries.

In the same way as we generated the phrase-substitutables, we can break
up the input query into segments, and replace one or several segments by
statistically significant related segments. This will help cover the
infrequent queries.

-   generate $m$ candidate whole-query substitutions
    $q_i\mapsto q_{i1}, q_{i2}\dots q_{im} $

-   segment query into phrases $p_1\dots p_n $

-   for each phrase $p_i $

    -   generate $k$ phrase$-$substitutions
        $p_i\mapsto p_{i1}, p_{i2}\dots p_{ik} $

    -   generate new query from a combination of original phrases and
        new phrases: $q_i \mapsto p_1\dots p_j^{\prime}\dots p_n$

more information[^1]

Query Rewriting Basing On Click Data
------------------------------------

### Problem Definition

-   We call the set of engine click set ${\mathcal}{L}$, which is
    consisted with tuples $\langle q, u, f_{qu} \rangle$. Where $u$
    represent the URL and the $q$ represent the query. What’s more,
    $f_{qu} $ is the number of times that the users issued query $q$ to
    the search engine and clicked on URL $u$. And
    ${\mathcal}{Q}\  \text{and}\  {\mathcal}{U}$ is the set if all
    queries and all URLs.\
    We will consider clock log ${\mathcal}{L}$ as a bipartite graph
    ${\mathcal}{G} = ({\mathcal}{Q}, {\mathcal}{U}, E)$, and
    $(q, u)\in E \text{ with weight} f_{qu}$.

-   And we define a *concepts* set
    ${\mathcal}{C} = \{c_{1}, c_{2}, \dots, c_{k} \} $. The element in
    ${\mathcal}{C}$ can be only one concept, such as “shoes” , in more
    complex cases we may have different classes in a taxonomy.

-   A *seed set*
    ${\mathcal}{S} \subseteq {\mathcal}{U}\times{\mathcal}{C}$, and the
    seed set ${\mathcal}{S}$ is consists of $\langle u, c\rangle$ pairs.
    We can regard this as label the URL with concept $c_{k}$.[^2]

### A Random Walk Algorithms

Intuitively the queries (and URLs) are related to the concept of shoes
because they are connected closely in the graph to the seed set that
represents this concept.

For some query $q \in {\mathcal}{Q}$ , we compute the affinity of $q$ to
some seed node $s \in {\mathcal}{S}$ as the probability that a random
walk that starts from $q$ ends up at node $s$. The affinity is the
probability that $q$ belongs to concept $c_{i}$ as seed $s$.

Note that in this walk the nodes in the seed set act as *absorbing
nodes*, that is, sink nodes in the state transition graph from which the
random walk cannot escape. The probability of absorbing nodes are
certainly $1$, which is easy to understand.

And the further away the queries and URLs from seed sets’ URLs, the less
related it should be to the URL’s class. We model this by introducing an
absorbing “null class” node $\omega$ to the graph.

Performing a random walk for every query in the graph is computationally
prohibitive for a large graph. However, there is a simple iterative
algorithm that can compute the class probabilities efficiently. We will
now describe the al- gorithm for the case that we have a single concept
$c$ (that is, ${\mathcal}{C} = {c}$), and then show how to generalize to
the case of multiple classes.

Let $\ell_q $ (or $\ell_u $ ) denote the random variable pertaining to
the concept label for query $q$ (or URL $u$). We want to compute
$P(\ell_{q} = c) $Let $\alpha$ be the probability of making a transition
to the null class absorbing node, from any node in the graph. Then we
have that:
$$P(\ell_q = c) = (1-\alpha) \sum_{u:(q,u)\in E}w_{qu}P(\ell_{u} = c) \eqno{(1)}$$
where $$w_{qu} = \frac{f_{qu}}{\sum_{u:(q,u)\in E}f_{qu}}$$ we have that
$P(\ell_u = c) = 1 $ if the pair $\langle u, c\rangle$ belongs in the
seed set, and zero otherwise

For all other URLs, the probability $P(\ell_{u} = c) $ is again
recursively computed as:
$$P(\ell_u = c) = (1-\alpha) \sum_{q:(u,q)\in E}w_{uq}P(\ell_{q} = c) \eqno{(2)}$$
where $$w_{uq} = \frac{f_{uq}}{\sum_{q:(u,q)\in E}f_{uq}}$$

\[H\]

\[1\] the seed set ${\mathcal}{S}$ for class $c$, the click-graph
${\mathcal}{G}$, the threshold parameter $\gamma$ , the transition
probability $\alpha$ to $\omega$ $P (\ell_{q} = c)$, for every query q
$P(\ell_{u}=c)=1$
$P(\ell_{q} = c)=(1-\alpha)\sum_{u:(q,u)\in E}\omega_{qu}P(\ell_{u} = c) $
$P(\ell_{q} = c) = 0 $
$P(\ell_{u} = c)=(1-\alpha)\sum_{q:(u,q)\in E}\omega_{uq}P(\ell_{q} = c) $
$(\ell_{u} = c) = 0 $ Output $P (\ell_{q} = c)$, for every query q and
class ${\mathcal}{Q} $

The algorithm generalizes naturally to the case where there are multiple
concepts, but this process is memory intensive since it requires to
maintain probability vectors for all $k$ classes. Furthermore, the
fraction of the graph that will be explored will increase substantially
even if we prune nodes with low probabilities, since the size of the
seed set may be significantly larger.

This problem can be addressed by considering the con- cepts one at the
time. Let ${\mathcal}{S} = {{\mathcal}{S}_1, \dots {\mathcal}{S}_k} $be
the partition of the seed set into the k concept. Apllying Algorithms 1
directly for each ${\mathcal}{S}_{i}$ is incoreect, because we ignore an
important information: ${\mathcal}{S}\backslash {\mathcal}{S}_i $ is
null class of ${\mathcal}{S}_i $. Apllying with this, we will explore
much smaller fraction of click graphy. we fix the probability of the
nodes in ${\mathcal}{S} \backslash {\mathcal}{S}_i$ to belong to the
class $c_i$ to zero, thus making them into null absorbing nodes.

\[H\]

\[1\] the seed set $S = \{S_{1},\dots,S_{k}\}$ for
concepts$C = \{c_{1},\dots,c_{k}\} $, the click-graph $G$ the threshold
parameter$\gamma $, the transition probability $\alpha$ to $\omega $
$P (\ell_{q} = c)$, for every query q and class c $P(\ell_{u}=c_{i})=0$
$P(\ell_{u}=c)=1$
$P(\ell_{q} = c)=(1-\alpha)\sum_{u:(q,u)\in E}\omega_{qu}P(l_{u} = c) $
$P(\ell_{q} = c) = 0 $
$P(\ell_{u} = c)=(1-\alpha)\sum_{q:(u,q)\in E}\omega_{uq}P(\ell_{q} = c) $
$(\ell_{u} = c) = 0 $ Output $P (\ell_{q} = c)$, for every query q and
class c

more information here[^3]

REMAIN TO BE SOLVED
===================

The problems:

1.  other algorithms in the paper, and the comparasion between them

2.  how to bid for AM

[^1]: Generating Query Substitutions: Jones et al, in Proc of WWW 2006

[^2]: we can also define the seed set as labeled queries or a mix of
    quries and URLs. For simplicity, we will restrict ourselves to the
    case that the seed set consists only of URLs.

[^3]: Using the Wisdom of the Crowds for Keyword Generation: Fuxman et
    al., In proc of WWW 2004
