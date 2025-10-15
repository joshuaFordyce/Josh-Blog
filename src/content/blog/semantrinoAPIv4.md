---
title: 'Agentic RAG for Complex Reasoning & Cost Control '
description: 'In this article, we explored adding innovative features to the semantrino API'
pubDate: 'December 8, 2025'
category: 'Data Engineering and Data Infrastructure'
heroImage: '../../assets/images/pexels-tranmautritam-326503.jpg'
tags: ['ML']
---

Agentic RAG for Complex Reasoning & Cost Control

This article details how we'll add innovative features to the SemanTrino API. We'll focus on using autonomous agents and intelligent caching to solve multi-step problems efficiently and reduce API costs.


When Simple RAG Fails

- Traditional RAG pipelines follow a fixed, linear path. This approach collapses when faced with complex, multi-hop queries or when dealing with the high latency and cost of external LLMS. We'll delve into improving the architecture in a way that helps us plan, execute, and optimizes autonmously.

- Innovation 1: Hierarchical, Multi-Agent Query Planning
  - We plan on moving beyond linear RAG by implementing a Query Planning Agent using a framework like LanGraph. this addresses the complexity challenge.
    - Decompostion
      - An initial, fast LLM acts as the Planner. It takes a single complex query and then decomposes it into discrete, sequential sub-tasks.
    - Hierarchical Execution
      - Each sub-task is executed sequentially by the RAG system. The result of the first retrieveal/generation step is automatically fed as context into the prompt for the next step
    - Synthesis
      - A final Synthesis Agent collects the sequential results to generate a single, comprehensive final SQL query.

- Innovation 2: Semantic Caching for Cost Optimization
  - To directly combat LLM API costs and latency, we want to implement a Semantic Cache Layer using a dedicated key-value store and vector embeddings
    - How it Works:
      - When a new query arrives, the system calculates it's vector embeddings, it then performs a vector similarity search against the cache keys
    - The Value:
      - If the new query is semantically similar to a previousone, the cached SQL is served instantly about hitting the external LLM API. This demonstrates sophisticated cost control and low-latency performance optimization

- Innovation 3: Corrective RAG for Automated Trust
  - We think it makes sense to  introduce Corrective RAG to validate the gnerated SQL query autonomously before committing to costly computation.
    - Critic Agent
      - A secondary, highly reliable Validation LLM is deployed. It is prompted to audit the generated SQl against the initial schema context, assiging a confidence score and providing specific crtiticism
    - Adaptive Action
      - If the confidence score is low, the system triggers an internal loop to refine the prompt based on the Critic's feedback , rerunning the generation step. This automated self-correction loop maximizes reliabilty before expensive execution

- Innovation 4: Hybrid Search
  - We think it makes sense to combine keyword search with vector searc. this ensures that we can retrieve tables that are both topically relevant and contain the exact column names speicified in the prompt
    - The Retriever
       - this part will run a vector similarity search and then run a second kyword search against the metadata.   
    - Reciprocal Rank Fussion
      - The results are merged using a technique like Reciprocal Rank Fusion to get a superior initial set of relevant tables/columns

- Innovatoin 5: Post-Retrieval Re-Ranking
  - We think that runnning a smaller more powerful re-ranknig model could help us refine our RAG's precision
    - Orchestrator
      - A amaller powerful re-ranking model is used to score the relevant of the documents/chunks against the original query
      - This secondary check removes "false positive" tables and ensures only the most contextually relevant schema elements are passed to the main, expensive LLM for SQL generation


- Innovation 6: Metadata-Augmented Multimodal RAG for Trino
  - This pushes our RAG into the advanced territory of using unstructured data as a source of metadata that influences the SQL generation
    - Ingestion
      - When indexing, chunks of the runboo/log file are created. Their metadata is enriched with tags
    - Query Flow
      - When a user asks, " Why is my sales query slow?", the system retrieves the most relevant Trino metadata and the relevant Performance Runbook chunks
    - Synthesis
      - The llm receivees the live schema and thhe best practice runbook steps simultaneously. The output isn't just the SQL; its the SQL plus a qarning/hint derived from the troubleshooitng docuemnts


These innovation and more are on the way for Semantrion version 2. Thank you for reading and I'll see you in my next article.
