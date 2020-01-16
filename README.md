# AutoSummarizzation
#####  text zuto summarization for news 
### 0 写在前
>>  <font size=16> &ensp;&ensp; 目前不论是大小云厂商还是提供SaaS服务的AI企业，大多推出了文本自动摘要这一功能。华为云去年之前还未做这个功能，但目前已经在EI里推出了这个功能。笔者在根据开课吧学习AI知识之后，将文本自动摘要采用无监督的方式进行实现，并且结合Flask进行项目展示。  
>>    &ensp;&ensp;自有文本摘要的研究以来，研究方法大致分为两大类，一种是生成式的文本摘要，另一种是抽取式文本摘要。对于中文训练集来说，由于尚且没有诸如英语文摘的训练集，所以很少能见到有关学者是采用生成式的方法进行中文的摘要训练；而对于英语生成式文本摘要代表作，可以查看[A Neural Attention Model for Abstractive Sentence Summarization](https://arxiv.org/abs/1509.00685)，引用原文一句```“Summarization based on text extraction is inherently limited, but generation-style abstractive methods have proven challenging to build. In this work, we propose a fully data-driven approach to abstractive sentence summarization. Our method utilizes a local attention-based model that generates each word of the summary conditioned on the input sentence. While the model is structurally simple, it can easily be trained end-to-end and scales to a large amount of training data. The model shows significant performance gains on the DUC-2004 shared task compared with several strong baselines.”```，可以看出深度学习方法的生成式存在着抽取式无法比拟的众多优势。笔者个人认为，生成式方法虽然目前存在一些尚未改进的地方（a.采用RNN结构，训练速度慢；b.深度学习框架不容易调测；c.深度学习框架依赖于训练集，训练集若集中于某个行业领域，则训练出的模型的普适性不高），但生成式方法一定是文本摘要的未来（a. 生成的句子更灵活，不用写一堆“蹩脚”的规则；b. 引入高层语义，更像人的思考之后说过的话）。  </font>
____
* 开始介绍本篇  
### 1 项目代码逻辑结构  
>> 在上代码之前，我们先思考几个问题：  
>>> （1）默认共识——摘要的句子都是这篇文本中重要的句子，那么怎么衡量这个句子的重要性？  
>>> （2）一篇文章总有一个主题，主题句在摘要中发挥的作用怎么体现？  
>>> （3）文章中的关键词做不做考虑，怎么找到关键词？  
>>> （4）如果上述的问题都解决了（问题1用PCA、问题2用LDA、问题3用TextRank），那么如何协调每种“摘要方法”之间的关系？分别配置的权重是多少？  
>>> （5）另外，结合项目针对的新闻领域，如果新闻本身是有标题的，那么标题是不是要考虑进去；并且输出结果还要符合新闻类文本摘要的一个基本要求，发生时间，人物（主人公）等需要考虑进去。  
>>> （6）笔者能够考虑的最后一点，需不需要设置字数限制？  

### 2 实现代码
____
* 当我们将上述的几个问题分别解决后，该项目也就完成的差不多了。  
____
&ensp;&ensp; 首先，我们先把每个句子进行一个多维的向量表征（主流使用使用词向量，可以是基于Word2vec，cove， glove，bert等一些主流的词向量训练方法），当然采用keras进行训练的话，也可以不用获取词向量，让其自动训练一个Embedding层即可，但本文采用的是已经训练好的模型（语料来源于中文wiki百科和搜狗新闻语料）[另外也可参考连接](https://github.com/Embedding/Chinese-Word-Vectors)，该种方法在深度学习上也叫采用预训练模型的方式。  

#### 2.1 如何表征一个句向量，并且怎么衡量该句向量对文章向量的重要程度？
&ensp;&ensp; 目前已经获取句向量的方法，常见的获取句向量的是加权句子包含词向量的思想，简单粗暴的有累加、平均、TF-IDF做权值和ISF加权。本文使用的是普林斯顿研究的方法，属于ISF方法，具体可以参考论文[A Latent Variable Model Approach to PMI-based Word Embeddings](https://arxiv.org/abs/1502.03520)，该论文17年发表，在有监督学习盛行的情况下，仍能曾获得业内的较高评价，可见方法可以获得较高的准确度。该方法需要在PCA的基础上求解协方差矩阵的第一特征向量作为句子向量的代表。具体PCA的原理，可以参考我的一篇[文章](https://github.com/CuiShaohua/NLP-/blob/master/%E6%89%8B%E5%86%99%E5%AE%9E%E7%8E%B0%E4%B8%BB%E6%88%90%E5%88%86%E5%88%86%E6%9E%90PCA.ipynb)。当然也可以借助sklearn上的PCA方法，但是需要注意的是，如果把文章想句子一样看做是一个特别长的句子，那么PCA计算协方差贡献率的时候需要进行更改，更改的部分贴在下方：
```Python
        # Get variance explained by singular values
        if n_samples != 1:
            explained_variance_ = (S ** 2) / (n_samples - 1)
        else:
            explained_variance_ = (S ** 2) / n_samples
            # explained_variance_ is zero
        total_var = explained_variance_.sum()
        #print(total_var)
        if total_var != 0:
            explained_variance_ratio_ = explained_variance_ / total_var

        else:
            explained_variance_ratio_ = np.ones_like(explained_variance_)
```
>> *  向量法盛行天下开始，在计算广告学等领域，推荐系统的本身是相似度和排序的使用，那么上述PCA将单个句子和单个文本都拿做一个向量来处置是十分恰当的，PCA降维之后，我们能够获得协方差矩阵第一特征向量，其在图上相当于做了某个方向的取向，句子向量与文本向量的余弦相似度越小，那么说明两者的取向相近，用这种似有监督却无监督的方法可以衡量每个句子对文本向量取向的贡献程度，也就是第一个问题解决了。  
#### 2.2 借助LDA求取文章的主题句
>> [LDA是什么？](https://zhuanlan.zhihu.com/p/31470216)， LDA是一种有监督的数据降维方法。我们知道即使在训练样本上，我们提供了类别标签，在使用PCA模型的时候，我们是不利用类别标签的，而LDA在进行数据降维的时候是利用数据的类别标签提供的信息的。 
从几何的角度来看，PCA和LDA都是讲数据投影到新的相互正交的坐标轴上。只不过在投影的过程中他们使用的约束是不同的，也可以说目标是不同的。PCA是将数据投影到方差最大的几个相互正交的方向上，以期待保留最多的样本信息。样本的方差越大表示样本的多样性越好，在训练模型的时候，我们当然希望数据的差别越大越好。否则即使样本很多但是他们彼此相似或者相同，提供的样本信息将相同，相当于只有很少的样本提供信息是有用的。样本信息不足将导致模型性能不够理想。这就是PCA降维的目标：将数据投影到方差最大的几个相互正交的方向上。而LDA是将带有标签的数据降维，投影到低维空间同时满足三个条件：尽可能多地保留数据样本的信息（即选择最大的特征是对应的特征向量所代表的的方向）。寻找使样本尽可能好分的最佳投影方向。投影后使得同类样本尽可能近，不同类样本尽可能远。具体实现代码可以参考我的一篇[文章]()。部分关键代码如下：  
```Ptthon  

```
### 2.3 关键词textRank  
>> 想必做NLP方向的各位同袍，应该对TextRank比较熟悉，TextRank可以看做是PageRank的2.0版本，而[PageRank](http://pr.efactory.de/e-pagerank-algorithm.shtml)可是谷歌发家致富的重要基石，通过爬取大量网页被引用指向，构成G引用矩阵，计算特征向量（每个元素代表一个网页的PageRank）$$ G=S*a+（1-a）*U*1/n $$


### 参考文献
[1]: Blei, D. M., Ng, A. Y., & Jordan, M. I. (2003). Latent dirichlet allocation. Journal of machine Learning research, 3(Jan), 993-1022.

[2]: Hofmann, T. (1999). Probabilistic latent semantic indexing. In Proceedings of the 22nd annual international ACM SIGIR conference on Research and development in information retrieval (pp. 50-57). ACM.

[3]: Li, F., Huang, M., & Zhu, X. (2010). Sentiment Analysis with Global Topics and Local Dependency. In AAAI (Vol. 10, pp. 1371-1376).

[4]: Medhat, W., Hassan, A., & Korashy, H. (2014). Sentiment analysis algorithms and applications: A survey. Ain Shams Engineering Journal, 5(4), 1093-1113.

