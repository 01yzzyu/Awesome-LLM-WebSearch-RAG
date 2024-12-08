**WeKnowRAG: An Adaptive Approach for RetrievalAugmented Generation Integrating Web Search and Knowledge Graphs**

1. **Motivation**
     **LLMs的问题与挑战**：大语言模型（LLMs）在推动自适应智能体发展方面有显著贡献，但存在关键问题，如易产生事实性错误信息和“幻觉”内容，影响其可靠性，这在实际应用场景中是严重阻碍。
   
     **RAG方法的局限性**：检索增强生成（RAG）方法虽能增强LLMs，但当前实现通常依赖密集向量相似性搜索进行检索，对于复杂查询存在不足，如受限于元数据范围、难以在相似向量空间块中回答复杂查询、检索效率低等。
   
     **知识图谱（KGs）的优势**：知识图谱能提供实体和关系的结构化、精确表示，可辅助RAG系统更精准检索信息，且能通过持续纳入新信息扩展，领域专家可构建特定领域知识图谱提供精确可信数据。


3. **Method**

 **Webbased RAG**
         **Web Content Parsing**：利用BeautifulSoup库解析原始HTML源代码，以理解非结构化数据、转换为结构化数据并获取回答问题所需信息。
         **Chunking**：采用token  level分块策略，将文档分为多个段落，并通过实验确定最佳分块大小。
         **Multi  stage Retrieval**：采用多阶段检索方法，第一阶段利用稀疏检索从页面结果块和片段块中收集候选，根据BM25分数选择前K个候选；第二阶段采用混合搜索，结合稀疏检索（BM25）和密集检索（基于bge  large  en  v1.5模型的嵌入相似度），先通过稀疏检索选择前M个候选，再通过密集检索选择前N个候选，最后使用bge  reranker  large模型重新排序，且添加页面名称和片段用于检索。
         **Answer Generation with Self  Assessment**：引入自我评估机制，让LLM指示生成答案的置信水平（高、中、低），仅当置信水平达到指定要求时接受答案，否则输出“我不知道”。

 **Knowledge Graphs**
         **Domain classification**：通过初始调用LLM对问题进行领域分类，对于电影、体育、金融和音乐四个领域，当模型确定性超过90%时进行分类，否则归为开放领域。
         **Query generation**：根据领域分类进入相应提示阶段，调用LLM返回结构化分析结果，查询生成器将其转换为与KG API兼容的结构化查询。
         **Answer Retrieval and Post  processing**：通过API对KG执行结构化查询检索候选答案，对于后处理问题应用额外推理，采用基于规则的系统结合机器学习技术处理时间推理、数值计算和逻辑推理。
     
**Integrated method**：根据CRAG数据集特点，分析各领域信息变化速度（静态、缓慢变化、快速变化、实时），提出自适应框架，智能平衡知识图谱和基于Web的RAG方法的使用。对于稳定领域优先使用KG工作流；对于信息逐渐变化的领域，在保持KG主导地位同时定期更新；对于实时性要求高的领域，更依赖基于Web的RAG工作流。


5. **Experiment**
     **Experimental settings**
         **Datasets**：离线测试使用CRAG数据集的200个问题子集，以快速迭代和优化；在线评估使用完整的CRAG数据集。
         **Evaluation metrics**：采用三分制评分系统（正确为1分、错误为  1分、缺失为0分），使用GPT  4进行基于模型的自动评估，评估指标包括准确率、幻觉率、缺失率和分数（准确率与幻觉率之差）。
         **Parameter settings**：基于Web的RAG第一阶段检索相关候选数K分别为200（页面结果块和片段块），第二阶段稀疏检索候选数M为5，密集检索候选数N为20；采用bge  large  en  v1.5作为密集检索模型、bge  reranker  large作为重新排序模型、llama  3  70b  instruct  awq作为Chat LLM模型。

     **Main results**
         **在线提交结果**：在KDD Cup 2024  CRAG Round 2任务3的最终提交中，展示了两个版本的在线自动评估性能，version2相对version1有三方面改进（增强开放领域提示处理、提高模型对错误前提查询的关注、调整相关提示），整体方法先利用KG流程查找答案相关信息，再用RAG流程合成最终答案。
         **离线测试结果**：通过设计本地测试集，逐步改进策略，包括分类提示、特定领域提示、非法问题优化、开放领域优化等，不断提高模型性能。
     **Ablation study**
         **Chunk size**：在RAG分块过程中，实验不同块大小对性能影响，发现块大小为500在最终提交中表现最佳。
         **Confidence level**：在答案生成的自我评估阶段，实验不同置信水平，结果表明置信水平为“高”时得分最高。


7. **Result**
     **实验结果表明**：WeKnow  RAG方法通过Web  based RAG工作流和KG工作流中各模块有效发挥作用，实现了良好性能。KG工作流通过函数调用从知识图谱中提取特定信息提供准确答案，Web  based RAG工作流通过多阶段检索从网页获取更多相关信息并通过自我评估减少幻觉。该方法在广泛的离线实验和在线提交中表现出色，证明了其有效性。
