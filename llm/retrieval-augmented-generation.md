# Retrieval Augmented Generation

RAG isn't a prompting technique, It's a whole LLM-based system.

**Definition:** Proposed by a group of Meta AI scientists [Lewis et al. (2021)](https://arxiv.org/pdf/2005.11401.pdf), Retrieval Augmented Generation (in short, RAG) is an LLM-based system that can access large amount of external knowledge sources.

_Mechanism:_ RAG is consisted of 2 components:

- Retriever: an information retrieval component.
- Generator: a text generator LLM (more specific, a chat model)

RAG takes a prompt and one or more sources as input:

1. Retriever vectorized the sources.
2. Retriever vectorized the query.
3. The query vector is matched against the sources vector for close matches.
4. The matching parts of the sources + The original query are sent into the Generator.
5. The Generator produces the final output.

There are a lot of different ways to vectorize the sources. Google's NotebookLM uses **Word2Vec**, a vectorized technology that has been pioneered by Google over a decade ago.

**Pros:**

- **Real-time update:** The external knowledge is updated real-time since the source can be changed easily.
- **High accuracy**: Improve reliability, more factual consistency, mitigate "hallucination" given a good vectorization technique.
- **No need retraining**: The Generator can be finetuned and its internal knowledge can be modified effectively without retraining of the model.

_Vendors:_

| Vendor            | Google NotebookLM |
| ----------------- | ----------------- |
| Max sources       | 50                |
| Source size limit | 200MB             |
| Source text limit | 500K words        |
| Total capacity    | 25M words         |
| Context window    | 1M tokens         |
