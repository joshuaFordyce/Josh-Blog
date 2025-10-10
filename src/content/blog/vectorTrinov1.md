---
title: 'Data-to-vector ETL '
description: 'In this article, we explored a specialized ETL pipelien for a RAG system '
pubDate: 'April 28, 2025'
category: 'Data Engineering and Data Infrastructure'
heroImage: '../../assets/images/2 - FA6Yb.jpg'
tags: ['ML']
---
Data-to-Vector ETL

The vectorTirno code implements a specialized ETL pipeline that focuses on preping enteprise data for a Retrieval-Augmented Generations system
- Extract:
    - Data and metadata are extracted from the Trino distributed query engine using the trinoConnect and TrinoMetadataRetriever classes
- Transform: 
    - The raw data and schema information are transforemd into structured text documents and then into high-dimensional numerical embeddings using the Vectorization class.
      This Vectorization class employs the SentenceTransformer
- Load:
    - The resulting vectors are loaded into the ChromaDB vector database


ChromaDB for Rag

- Chroma is perfect for Rag applications for a couple of reasons
  - It's python native which allows developers like me to use it directly without complex driver configuration
  - For initial development, we can run chroma entirely in memory
  - For staging and production, we can transition our chroma instance to persitent storage thats backed by a disk directory or a client server mode. this ofers the necessary durability a production application would need
  - Chroma allows for metadata filtering which means we can store structured metadata alongsied each vector
    - this enables hybrid search meaning that we can perform a semantic search (vector similarity) and then filter the results using specific Trino knowledge.
      This makes sure the LLM receives highly targeted context
CLI Pattern
We used the click library to define the run_pipeline function that dictates the deployment pattern. Lets go over some key reasons
for our choice of command-line tool for this microservice. 

Here are some key features of Click that made it perfect for the vectorTrino microservice.
- Click is great or organizing large, multi-functional apps into intutitive hierarchies. The nesting of @click.group and  @group.command commands allowed me to follow a predictable noun-verb hierarchy
- The aforementioned commands are python decorators and that helps us keep the code highly readable and maintainable

Here are the Key features of the application that we used Click for.

- Batch Processing:
  - This was ideal for me since the focus was less on low latency and moreso on data completelness for our scheduled tasks

  MLOps Orchestration
  - In future integrations of the semantrino, we may want to have the cli executed by a workflow orchestrator like Apache Airflow. The CLI setup ensures that we can easily switch from a manual setup to a automated one

  configuration management
  - The click.argument('config_file'') pattern does the dual job of enforcing all connection details and operation aprameters are exetnralized in a config file, while making it simple for the user to define dynamic parameters.
    This allows us to keep the code environnment agnostic

Thanks for reading along with us in the first part of our deep dive into the architecture of the vectorTrino tool. I'll see you in part two!


    
