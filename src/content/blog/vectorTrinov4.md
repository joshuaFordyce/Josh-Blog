
Building a Robust Batch System: State Management and Error Handling

This is the fourth article in this series and it focuses on analyzing the design decisions related to global state, modulei mports and error handling. This focuses on making the batch process reliable


GlobalState: The Efficiency/Robustness Trade-off

I made the decision to implement a basic Singleton Pattern using global variables within the get_retriever and get_vector_store functions. I wanted to prioritize initial batch performance over architectural elegance

The core benefit is Resource Conservation. Establishing connections to eternal services whether its authenticatin with Trino or initializing a ChromaDB client involves signifcant I/O and latency.

- By ensuring the initialization happens only once per script exwecution, i eliminate redundant connection overhead. This is a crucial efficiency gain
  for a CLI intended to run as a short-lived batch job where startup time directly impacts throughput

- MLOPs Caveat
  - While this is a suitable design pattern for single-threaded batch script, reliance on global state can introduce untestability and concurrency issues
    - If we later adapt this script to run parallel ingestion threads using multiprocessing, the global variables may become a source of race conditions, leading to unpredictable errors and data corruption
    - In a true production system, we could refactor to use a dedicated dependency injeciton framework. A DI container maintains the singleton isntance while gauranteeing thread-safety and easy mocking for unit tests.
I/O and Error Handling: Resource Gaurantee

I implemented try..except...finally blocks in the trinoConnect methods to prevent resource leaks

Preventing Connection Starvation
- The purpose of the finally block is simple
  - Database Connections are Assets
    - Database Engines manage a limited pool of connections. When a batch script fails, the script might abruptly exit.
  - The Critical Action
    - The finally block ensures that the essential cleanup step is executed even if an exception occurs
  - Impact
    - By releasing the connection back to the Trino Pool, we prevent connection starvation, guaranteeing that other critical systems can access the databases without being blocked by a failed ingestion script
CLI Entry Point: Containerizationa nd Orchestation
The use of the python idiom if __name__ == '__main__': run_pipeline() is what makes the VectorTrino ingestion script truly deployable and suiable for modern cloud infra
- Docker integration
  - this structure makes the script a perfect target for the CMD instruction in a Dockerfile. The script can be executed as a standalone, isolated process, easily managed by a container orchestration platform like k8s
- Orchestrator Integrations
  - It provides a clean, predictable entrypoint for orchestrators like Airflow or Kubeflow
 
Thanks for reading and I'll see everyone in our next and final portion of this series.
