---
title: 'RAG Core: Orchestration, Resilience, and Resource Management'
description: 'In this article, we explored the core of our RAG API'
pubDate: 'November 24, 2025'
category: 'Data Engineering and Data Infrastructure'
heroImage: '../../assets/images/pexels-pixabay-355952.jpg'
tags: ['ML']
---
The RAG Core: Orchestration, Resilience, and Resource Management

The Hybrid RAG pipeline: Langchain Orchestration

The biggest trap I wanted to avoaid was trying to custom-code the entiere RAG workflow. Instead of building a brittle, custom sequence, I leveraged the Langchain framework. 

- LangChain as the Assemply LIne Abstraction
  - Langchain is the abstractino layer that turns the individual components(the database, the llm, the parser into a coherent pipeline.
  - Our system is composed of two primary, standardized modules
    - The Retriever(DataConnecor)
      - This module is responsible for fetching external context. We built a custom Langchain Retriever that connects directly to our knowledge base. It doesn't care if the underlying storage is ChromaDB, Pinecone or another database. It's only focus is about getting a list of relevant documents.
    - Generator (LLM Interface)
      - This modules takes the rerieved context, structures it into a final prompt and calls the LLM. This separation allows us to swap out the LLM provider later without ever modifying the retrieval logic
Why LangChain Works Percectly with ChromaDB
- Standardized Document Format
  - Langchain uses the simple Document object structure for all information retrieval. Our  custom retriever is designed to take the raw search results from the database and convert them into LangChains standardized Document format. This converion makes sure that every piece of knowledge, looks the same to the Generator module
- Decoupled Knowledge Store
  - ChromaDB serves a highly efficient high-performance vector store and LangChain abstracts the process of indexing and querying this store. This emans that the main orchestration logic of our API doesn't need to contain any ChromaDB-specific code. If we decide to migrate to a large distributed vector store like weaviate, we'd only have to swap out the Retriever class. This allows us to retain modularity and maintanability

Resource Management: The Single Trade-Off

- The trino connection and vector store client are expensive, stafuel resources. Because of this, re-initializing them for every incoming API request would introduce significant latency and quickly lead to resource exhaustion.
- To avoid this, we employed the Singleton Pattern via a global variable. This gaurantees that the connection object is created only once per process. While global state is usually discouraged, this trade-off is necessary for resource conservation and achieving low latency

Context Augmentation: Structuring the Prompt

- Retrieval
  - The LangChain Retriever fetches the most semantically relevant metadata
- Structuring:
  - The agent formats this data into a clear, predictable structure
- Context Injection
  - This structured schema is injected directly into the context window of the LLM prompt. This forces the LLM to ground its SQL generation in the provided schema, acting as a powerful technical guardrail that drastically reduces LLM halluciantions and ensures the outputted SQL is runnable against the customer's live database

In this article we detailed the stratechic architecture of the semantrino RAG Core. We covered Langchain as a resilient layer, the SIngleton Trade-OFf via a global variable and LLM guardrail constructed from the retrieved Trino Schema.
Thank you for reading and I hope to see you in part 3.
