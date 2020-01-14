# AutoSummarizzation
#####  text zuto summarization for news 
### 0 写在前
>>    目前不论是大小云厂商还是提供SaaS服务的AI企业，大多推出了文本自动摘要这一功能。华为云去年之前还未做这个功能，目前在EI里已经做了这个功能。笔者在根据开课吧学习AI知识之后，将文本自动摘要采用无监督的方式进行实现，并且结合Flask进行项目展示。  
>>    自有文本摘要的研究以来，研究方法大致分为两大类，一种是生成式的文本摘要，另一种是抽取式文本摘要。对于中文训练集来说，由于尚且没有诸如英语文摘的训练集，所以很少能见到有关学者是采用生成式的方法进行中文的摘要训练；而对于英语生成式文本摘要代表作，可以查看[A Neural Attention Model for Abstractive Sentence Summarization](https://arxiv.org/abs/1509.00685)，引用原文一句```“Summarization based on text extraction is inherently limited, but generation-style abstractive methods have proven challenging to build. In this work, we propose a fully data-driven approach to abstractive sentence summarization. Our method utilizes a local attention-based model that generates each word of the summary conditioned on the input sentence. While the model is structurally simple, it can easily be trained end-to-end and scales to a large amount of training data. The model shows significant performance gains on the DUC-2004 shared task compared with several strong baselines.”```，可以看出深度学习方法的生成式存在着抽取式无法比拟的众多优势。笔者个人认为，生成式方法虽然目前存在一些尚未改进的地方（a.采用RNN结构，训练速度慢；b.深度学习框架不容易调测；c.深度学习框架依赖于训练集，训练集若集中于某个行业领域，则训练出的模型的普适性不高），但生成式方法一定是文本摘要的未来（a. 生成的句子更灵活，不用写一堆“蹩脚”的规则；b. 引入高层语义，更像人的思考之后说过的话）。  










### 参考文献

[1] https://arxiv.org/abs/1509.00685

