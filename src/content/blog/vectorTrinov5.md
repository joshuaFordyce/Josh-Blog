MLOps Future-Proofing and PipelineExtensions


This final part dicusses how th ecurrent structure prepares the service for advanced mlops features and ncessary future extensions

Scaling the Vectorization: Achieving Petabyte Throughput

- The single-threaded batch job is perfect for our current needs, but as we try to scale the application to handle enteprise data theres a bottleneck that wee need to adress

- Horizontal Scaling (CPU-Bound Work):
  - The solution for petabyte-scale data ingestion is parallelizing the _get_relevant_documents loop. This is achieved by shifting the workload
    onto multiple worker processes using libraries like Python's multiprocessing or more robustly in a containerized environment. The work would be
    divided by Trino catalog or table, allowing hundres of cores to perform embedding generation concurrently, dramatically cutting down processing time

- Asynchronous I/O (I/O-Bound Work):
  - While syncrhonous now, converting the Trino queries and ChromaDB insertion calls to be non-blocking using asyncio would optimize throughput.
  - If the system is waiting for Trino to return a large dataset, an asynchronous client can concurrently initiate the next wuery or perform other I/O, preventing the processor from sitting idle


Extending the Pipeline for LLMOPS

- The clean functional decoupling built into the current architecture creates ideal extension points for embedding necessary LLMOps and data governance features

- Data Quality Check
  - This helps us gauranteed that low-quality data never enters the vector store. This is the first line of defense agains RAG halucinations rooted in bad context
- PII Redaction Service
  - Before any sensitive content is vectorized and persisted the service injects a masking function into the transformation pipeline and this ensures compliance without altering the core ingestion logic
- Model Registry Integration
  - Removes the hardcoded model  name. Instead, the class queries a central registry, fetching the latest approved versinoof the embedding model. This allows the MLOps team to update or replace the embedding model without touching the vectorTrino code

This is the last article in this series about the VectorTrino Batch service. Once we finish local testing and make the application fully available, we'll begin working on vectorTrino version 2.0
