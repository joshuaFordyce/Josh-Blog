---
title: 'Architecture of Decoupling: Trino, ChromaDB, and Data Abstraction '
description: 'In this article, we explored how classes abstract data retrieval'
pubDate: 'April 28, 2025'
category: 'Data Engineering and Data Infrastructure'
heroImage: '../../assets/images/2 - FA6Yb.jpg'
tags: ['ML']
---
Architecture of Decoupling: Trino, ChromaDB, and Data Abstraction

This is the second part of this series. In this article, we'll dive into how the classes abstract the complexity of data retrieval and vector databases operations, enhancing modularity and maintainability


Data Retrieval Layer: TrinoMetadataRetriever

THe TrinoMetadaaRetriever serves as the Data Access Object for the data warehouse. A DAO is a design pattern that is used to separate the data persistence logic(how dat is stored, retrieved,an dupated) from the business logic( what the app does with the data).
The DAO acts as a gatekeeper betwne the application's core code and the database. 

- By inheriting from langchain_core.retrievers.BaseRetriever, the TrinoMetadataRetriever class conforms to a standardized einterface. This makes the pipeline interoperatble with other components in the LLM ecosystem
  - Any langchain component can instantly consume data form the TrinoMetadataRetriever without needing to know the implementation of the rest of the code
  - The BaseRetriever interface mandates that the retriever output Documents, ensuring predictable context deliver to the LLM generation phase
  - If we ever decide to replace Langchain with LlamaIndex or replace ChromaDB with pinecone, the application logic should only need a minior update to handle the new client.
- The class has the dual responsibility fo fetching two types of documents
  - Metadata:
    - Database schema inforation which enables the llm to answer questiosn about the structure of the data
  - Raw data:
    - The actual row level data from the tables which allows the llm to answer questions about the content

- The TrinoConnect class encapsulates all database-specific logic whihc shields the main retriever logic from changes in the
  Trino driver or connection parameters

Vector Store Layer: Vectorization

The Vectorization class is a single, defined contract for the entire semant search process. It focuses on ensuring that the process of transforming text into search vectors is consistently handled

- Connecting the Embedding Model and ChromaDB client
  - The connection between the embedding model and the vector database happens at the moment of ChromaDB initilaization. THe vecotrization class manages both components but we explicitly delegate the embedding task to the configured model.
    - Embedding Model
      - Preselects all-MiniLM-L6-v2 as the embedding model, ensuring consistency across all generated vectors
    - VectorDB Client
      - The configure_chroma_client handles connection details and setup
The separation of the Retriever and Vectorization logic is a cornerstone of RAG modular design, allowing each compnent to be swapped out with minimal impact to the other
Thanks for reading and I'll see you in part 3.
