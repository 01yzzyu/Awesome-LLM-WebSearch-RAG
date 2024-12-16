# DRAGIN: Dynamic Retrieval Augmented Generation based on the Information Needs of Large Language Models

用于根据LLM文本生成过程中的信息需求来决定何时检索以及检索什么

优化对预存知识库的动态调用，而非提供基于实时网络搜索的生成能力。

![image.png](DRAGIN%20Dynamic%20Retrieval%20Augmented%20Generation%20base%2015dcd9a0c43a80a3b92fddfd32d1a036/image.png)

![image.png](DRAGIN%20Dynamic%20Retrieval%20Augmented%20Generation%20base%2015dcd9a0c43a80a3b92fddfd32d1a036/image%201.png)

DRAGIN 是一个轻量级的动态检索增强生成框架，它通过两个模块实现功能：

•	**RIND（实时信息需求检测）：** 实时检测 LLM 的信息需求，决定检索时机。

•	**QFS（基于自注意力的查询生成）：** 动态生成检索查询以满足生成过程的实时信息需求。

- **RIND（实时信息需求检测）：实时检测 LLM 的信息需求，决定检索时机**
    
    **核心原理：**
    
    RIND 模块通过综合评估以下三个因素，为生成中的每个 token 分配一个信息需求评分 (**SRIND**)，并基于评分判断是否需要触发检索：
    
    1.	**生成的不确定性（Uncertainty）：**
    
    •	使用熵 (**Entropy**) 衡量当前生成 token 的不确定性。熵越高，表示模型对该 token 的预测越不确定。
    
    •	计算公式：
    
    其中  是 LLM 在生成位置  对词汇表  中每个 token 的概率分布。
    
    2.	**对上下文的影响力（Token Impact on Context）：**
    
    •	通过自注意力机制评估当前生成 token 对后续 token 的影响。
    
    •	最大注意力值 (**amax**) 用来表示一个 token 对后续生成的影响程度：
    
    3.	**语义贡献（Semantic Contribution）：**
    
    •	使用二值化的语义指示器来过滤停用词，只关注具有语义意义的 token。
    
    **综合评分计算：**
    
    RIND 模块为每个 token 计算综合评分，超过预设阈值时，触发检索模块。
    
- **QFS（基于自注意力的查询生成）：动态生成检索查询以满足生成过程的实时信息需求**
    
    QFS 利用 Transformer 模型中的自注意力机制，从整个上下文中提取最相关的内容来构建检索查询。
    
    1.	**选择触发位置：**
    
    •对于 RIND 模块生成的 token 时触发检索需求，QFS 模块将生成该 token 时的注意力分布。
    
    2.	**提取相关性高的 token：**
    
    •从 Transformer 的最后一层提取自注意力权重：
    
    •按权重值降序排列，选择前  个权重最大的 token。
    
    3.	**还原选定 token 的文本顺序：**
    
    •将选定的 token 按照它们在原始上下文中的**顺序排列**，确保查询语义连贯。
    
    4.	**构造检索查询：**
    
    •将选定 token 拼接成检索查询，用以反映模型当前的知识需求。
    

BM25 作为检索模型

**评估数据集**

DRAGIN 的评估实验选用了 4 个知识密集型文本生成任务的数据集，这些数据集涵盖了多跳推理、阅读理解和常识推理等任务，测试模型在不同任务场景下的表现。

**1. 2WikiMultihopQA**

•	**任务类型：** 多跳问答（Multihop Question Answering）。

•	**目标：** 回答需要结合多个事实推理的问题。

•	**示例问题：**

“When did the director of the film Hypocrite die?”

•	答案需要先找到导演是谁，然后找到导演的去世时间。

•	**数据来源：** Wikipedia。

•	**配置：**

•	使用 BM25 作为检索模型，每次检索返回 3 个文档。

•	训练过程中使用 6 个示例作为学习上下文。

**2. HotpotQA**

•	**任务类型：** 多跳问答（Multihop Question Answering）。

•	**目标：** 回答需要多个证据跨越不同段落的问题。

•	**示例问题：**

“What film directed by Brian Patrick Butler was inspired by a film directed by F.W. Murnau?”

•	答案需要先找到由 Butler 导演的电影，然后确认这部电影受到 Murnau 导演作品的影响。

•	**数据来源：** Wikipedia。

•	**配置：**

•	使用 BM25 作为检索模型，每次检索返回 3 个文档。

•	训练过程中使用 8 个示例作为学习上下文。

**3. IIRC（Incomplete Information Reading Comprehension）**

•	**任务类型：** 不完整信息阅读理解（Incomplete Information Reading Comprehension）。

•	**目标：** 回答需要根据多个文档的上下文补全信息的问题。

•	**示例问题：**

“What is the age difference between the kicker and the quarterback for the Chargers?”

•	答案需要分别找到踢球手和四分卫的出生年份并计算其年龄差。

•	**数据来源：** Wikipedia。

•	**配置：**

•	使用 BM25 作为检索模型，每次检索返回 3 个文档。

•	数据集中包含 954 个问题（已移除无答案的问题）。

•	使用 8 个示例作为上下文。

**4. StrategyQA**

•	**任务类型：** 常识问答（Commonsense Question Answering）。

•	**目标：** 回答需要隐含推理策略的问题。

•	**示例问题：**

“Is it common to see frost during some college commencements?”

•	答案需要推理出大学毕业典礼的时间范围以及可能的天气条件。

•	**数据来源：** Wikipedia。

•	**配置：**

•	使用 BM25 作为检索模型，每次检索返回 3 个文档。

•	使用 8 个示例作为上下文。

**评估指标**

评估指标主要分为以下几类，用于量化模型在生成任务中的性能表现：

**1. 准确率（Exact Match, EM）**

•	**定义：** 预测答案与参考答案是否完全一致。

•	**意义：** 测试生成答案的精确性。

•	**计算方式：**

•	对于每个问题，若生成答案完全匹配参考答案，则记为 1，否则记为 0。

•	所有问题的平均分数作为最终的 EM 分数。

**2. F1 分数**

•	**定义：** 考虑答案的精确性（Precision）和召回率（Recall）的加权平均分数。

•	**意义：** 测试生成答案是否包含正确的关键词（即部分匹配的答案也能获得分数）。

**3. 准确率（Accuracy）**

•	**定义：** 对于二分类问题（如 StrategyQA 中的 Yes/No 问题），预测答案是否与参考答案一致。

**4. 精确率（Precision）**

•	**定义：** 生成答案中的正确部分占生成答案的比例。

**5. 召回率（Recall）**

•	**定义：** 生成答案中的正确部分占参考答案的比例。