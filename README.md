## OpenConcepts

[![License](https://img.shields.io/github/license/OpenKG-ORG/OpenConcepts?style=flat-square)](https://github.com/OpenKG-ORG/OpenConcepts/blob/master/LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/OpenKG-ORG/OpenConcepts?style=flat-square)](https://github.com/OpenKG-ORG/OpenConcepts/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/OpenKG-ORG/OpenConcepts?style=flat-square&color=blueviolet)](https://github.com/OpenKG-ORG/OpenConcepts/network/members)

OpenConcepts (http://openconcepts.openkg.cn/) 是一个基于自动化知识抽取算法构建的中文概念图谱，由浙江大学知识引擎实验室贡献。本次开源了OpenConcepts中的440万概念核心实体，以及5万概念和1200万实体-概念三元组，并提供json,ttl, json-ld多种格式的原始Dump下载。

概念是人脑对事物的本质反应，能够帮助机器更好的理解自然语言。相较于传统的知识图谱，OpenConcepts包含大量中文细粒度概念，且具备自动更新、自动扩充的能力。比如对于“刘德华”这一实体，OpenConcepts不仅包含“香港歌手”、“演员”等传统概念，还具有“华语歌坛不老男歌手”、“娱乐圈绝世好男人”等细粒度标签。

## OpenConcepts构建

构建知识图谱具有诸多挑战。早年的英文知识图谱如CyC、WordNet以及中文知识库如HowNet等大多通过专家手工构建，其构建成本非常高昂。Openconcepts采取完全自动化构建的方式，基于海量的中文网页数据和若干开放的中文知识库,通过自动化信息抽取、短语挖掘等自然语言处理技术，实现概念知识图谱的自动化构建。相较于传统的概念知识图谱，OpenConcepts的特点在于：

1. OpenConcepts包含大量的中文细粒度概念，这部分细粒度概念填补了中文细粒度知识的空白。

2. OpenConcepts是基于全自动化构建的方式，其整合了诸多自然语言处理算法并形成一套完整的知识抽取框架，具备自动化抽取、自动化扩展、自动化更新的能力。

OpenConcepts的自动化构建主要分为两大模块，1）概念知识的自动化抽取 2） 概念知识的融合，相关技术已经发表在[国际顶会KDD 2021](https://mp.weixin.qq.com/s?__biz=MzIwMTc4ODE0Mw==&mid=2247530031&idx=1&sn=8628218cbf4386a2ff667305d3d8d3cd&scene=21#wechat_redirect)。我们首先通过开放的知识库、百科InfoBox等结构化、半结构化数据抽取粗粒度的概念。对于细粒度的概念，我们采取短语挖掘和序列标注相结合的策略，通过实体-概念模板和无监督短语挖掘构造弱监督样本，并基于迭代的降噪学习训练基于序列标注的[概念抽取模型](http://openconcepts.openkg.cn/concept_extract_page)，在离线测试集上概念抽取模型准确率可达0.89，召回率可达0.85。然后，我们对抽取到的不同的实体和概念进行融合，并通过贝叶斯估计过滤掉低置信度的概念。此外，我们也构造人工规则约束对高层次的概念进行人工干预，保证准确率。


具体的说，我们首先从包含噪声的海量开放语料中提取常见的细粒度概念，然后获取候选概念和实例，并通过概率推理和概念匹配将候选概念和实例与相应的概念联系起来。我们定义了一组精准的模板来从高置信度的匹配查询中利用Bootstrapping方法提取概念短语。例如，“十大XXX”是一种可用于提取种子概念的模式。基于这种模式，我们可以抽取出“十大手机游戏”等概念。然而由于文本中存在大量的噪声，因此我们采用一种基于对齐一致性的Bootstrapping方法来处理含噪文本。假设在某一轮中找到的新模板p，n_s是现有种子概念集合中的概念数，p可以从查询集Q中提取这些概念。设n_e是p可以从Q中提取的新概念的个数，我们通过函数 Filter(p): 1) 和2)  维护模板集p ，其中、以及 是预定义的阈值，用于控制提取概念的精度。其次，我们通过对齐一致性对挖掘出的概念进行过滤，以提高细粒度概念的质量。最后，我们对挖掘出的细粒度实例和候选概念，利用概念判别器来判断每一个候选是概念还是实例，并通过概率推理和概念匹配将实例与这些分类的概念联系起来。具体的流程如下图所示：


针对长尾概念，我们通过短语挖掘和自训练从有带噪的搜索日志中提取长尾概念。我们首先基于短语挖掘算法，并利用外部领域知识图谱中的术语进行长尾的概念挖掘。具体来说，我们首先过滤停止词，然后使用现成的短语挖掘工具AutoPhrase在无监督的情况下对语料库进行短语挖掘。

我们同时采用了一种基于自训练的序列标注算法，用于长尾概念的挖掘，进一步提取一些分散的概念。具体而言，我们基于上述无监督方法生成的一些实例/概念作为种子训练一个CRF序列标注模型，并基于海量无标注数据生成大量弱监督伪样本，然后我们基于伪标注样本训练了一个基于BERT序列标注模型。之后，我们基于CRF、BERT和领域字典得到的预测结果进行一致性校验，以过滤掉伪标注样本中的噪声，并迭代生成更可靠的训练样本。

最后，我们将部分概念与预定义的同义词词典对齐。然后，我们通过通过每天的用户搜索实例热度计算置信度得分，并根据用户的点击行为来估计概念置信度分布。最后，我们将两个不同粒度的置信度得分联合构建实例-概念分类。


## OpenConcepts规模和用途

 本次，我们开源了OpenConcepts中的440万概念核心实体，以及5万概念和1200万实体-概念三元组。这些数据包括了常见的人物、地点等通用实体。我们的数据还在不断更新中。本次开源的数据可在openkg.cn 获取，OpenConcepts能够为智能推荐、智能问答、人机对话等应用提供数据支持。


我们的API接口开放在url{http://openconcepts.openkg.cn/api_page}。可以指定实例，返回若干概念实例集，包含相应的概念层级和置信度。例如我们可以发送http://openconcepts.openkg.cn/api/entity_concept?entity=南华园，来询问“南华园”这一实例。返回一个字典，其中"位置"概念属于概念级别1，置信度为0.43；"楼盘"属于概念级别2，置信度为0.75；"房产"属于概念级别1，置信度为0.43；"学校"属于概念级别2，置信度为0.25；"教育"属于概念级别1，置信度为0.14。同时在网站的分页面我们也开放了各类概念数据集的下载链接。


## 致谢


由于OpenConcepts是基于自动化算法从互联网语料获取的，其中难免存在错误的数据，在此表达歉意。感谢毕祯提供算法上的支持、唐坤实现在线演示系统，邓淑敏、余海阳、叶宏彬、杨嘉诚、李娟、邓鸿杰、李泺秋、杨帆、陈想、谢辛等提供数据和技术上的支持。

## 引用

```bibtex
@inproceedings{zhang2021alicg,
  title={AliCG: Fine-grained and Evolvable Conceptual Graph Construction for Semantic Search at Alibaba},
  author={Zhang, Ningyu and Jia, Qianghuai and Deng, Shumin and Chen, Xiang and Ye, Hongbin and Chen, Hui and Tou, Huaixiao and Huang, Gang and Wang, Zhao and Hua, Nengwei and Chen, Huajun},
  booktitle={KDD},
  year={2021}
}
```

## References

[1] ERNIE 2.0: A Continual Pretraining Framework for Language Understanding

[2] ERNIE: Enhanced Language Representation with Informative Entities
