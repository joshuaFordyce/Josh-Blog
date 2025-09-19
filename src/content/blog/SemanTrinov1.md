---
title: 'SemanTrino: RAG for Data '
description: 'In this article, we introduce the SemanTrino system'
pubDate: 'April 28, 2025'
category: 'Data Engineering and Data Infrastructure'
heroImage: '../../assets/images/2 - FA6Yb.jpg'
tags: ['ML']
---


Lately I'v been very interested in building applied AI systems. My journey is about more than just writing code; it's about solving real-world, high-stakes problems for clients. That's why I'm building Semantrino, an open-source project designed to tackle a common and frustrating issue for data analysts and engineers: the inability to easily discover and understand data within a large-scale data lake.

The Problem: A Data Lake You Can't Talk To 
Modern enterprises store vast amounts of data in distributed systems like Trino, a distributed SQL query engine designed for querying massive datasets. Trino makes data accessible, but it doesn't make it understandable. A data analyst might know they need "customer order data," but they have no way of knowing where that data lives within the thousands of schemas and tables in a data lake. The process of finding the right data is a manual, tedious, and frustrating one. My solution is a semantic search tool that lets you find what you're looking for by asking a simple question, just like you would with Google.

The Solution: A RAG Pipeline for Data 
Semantrino is a Retrieval-Augmented Generation (RAG) pipeline. A RAG pipeline combines the power of a large language model (LLM) with a custom knowledge base, ensuring the LLM's responses are grounded in accurate, up-to-date information. In my case, the knowledge base is the metadata and schema of the data lake.

Here's how the architecture works:

Ingestion: I connect to a Trino cluster and retrieve its schema—the catalogs, schemas, tables, and columns—along with their data types. I also pull small samples of raw data from key tables.

Vectorization: Using a sentence-transformer model, I convert this information into vector embeddings. These vectors are numerical representations that capture the semantic meaning of the text. For example, the vector for "customer_id" would be semantically close to "cust_id," even though the words are different.

Vector Store: These vectors are then stored in a vector store (ChromaDB), which is optimized for fast similarity search. It allows me to query the database using a vector and retrieve the most relevant vectors.

Retrieval: When a user asks a question like "Which table has customer order information?", Semantrino converts the question into a vector. It then queries the vector store for the most semantically similar pieces of information. This process retrieves the relevant table names and metadata.

Generation: Finally, the retrieved context is passed to an LLM, which uses this information to formulate a coherent, natural-language response, such as "Customer order information is available in the tpch.sf1.customer table."

The Workflow: A Multi-Part System 
Semantrino is designed as a series of microservices to ensure modularity and scalability.

The TrinoMetadataRetriever: This service is responsible for connecting to the Trino cluster and ingesting the metadata and sample data. It's the first step in building the knowledge base.

The VectorTrino Microservice: This is the heart of the system. It orchestrates the entire process: taking the raw data from the retriever, vectorizing it, and writing it to the ChromaDB vector store.

The QueryProcessor: This is the front-facing service that takes a user's natural language query, processes it through the RAG pipeline, and provides a final answer.

My next article in this series will focus on the VectorTrino microservice. I'll dive deep into the technical challenges I faced in building it, from handling abstract classes and Pydantic validation to tackling common typos and distributed system errors. It's a journey that has taught me invaluable lessons and is a major milestone in my transition from a systems support role to a forward-deployed engineer who can build and troubleshoot complex AI systems in the field.
