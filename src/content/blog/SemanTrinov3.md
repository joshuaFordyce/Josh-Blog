---
title: 'SemanTrino: A harrowing troublshooting  journey'
description: 'In this article, we the explore the process of testing the VectorTrino microservice'
pubDate: 'November 28, 2025'
category: 'Data Engineering and Data Infrastructure'
heroImage: '../../assets/images/2 - FA6Yb.jpg'
tags: ['ML']
---


My troubleshooting journey began with a fundamental debugging puzzle, a TypeError that wasn't about my code, but about its contract with the framework.

Confronting Abstract Classes: 

- The first error was a seemingly obscure TypeError: cannot instantiate abstract class. I knew this wasn't a simple typo. As an aspiring applied AI engineer, I had to recognize the pattern: my class was a subclass of LangChain's BaseRetriever, an Abstract Base Class (ABC). An ABC is a blueprint; it mandates that any class inheriting from it must implement specific methods. I was trying to run my code before fulfilling that contract by implementing _get_relevant_documents.

The Milestone: 

- This was my first major lesson in framework-first development. You don't just write code; you build on the shoulders of giants. The key is understanding and adhering to the framework's architecture. My fix was to implement the required method, making my retriever a valid component within the LangChain ecosystem.

Data Schema and Pydantic:

- The next hurdle was a TypeError related to my class's __init__ method. It was a classic "trap" from Pydantic, the data validation library that underpins much of LangChain. My BaseRetriever class was a Pydantic BaseModel, which enforces a strict schema. The error was a result of passing custom, undeclared arguments to the super().__init__ call.

The Milestone: 
- I learned the critical importance of data contracts in modern AI systems. Pydantic taught me to be explicit about my data. My solution was to declare all public attributes as Pydantic fields and handle internal, non-serializable objects (like the Trino client) using PrivateAttr. This approach made my code more robust, readable, and ready for deployment in a production environment where data integrity is paramount.

Part 2: 
The Path to Applied AI Systems
- With the core architecture in place, the next phase of the project was about making it work in the real world. This is where the applied part of "applied AI" comes in. The errors were no longer about fundamental OOP but about real-world integration.

The Typos of the Real World 
- I ran into a series of errors that were frustratingly simple but incredibly instructive. My chosen embedding model, all-MiniLM-Lgv2, was giving a 401 Client Error. A quick search revealed a typo; the correct model was all-MiniLM-L6-v2.

The Milestone:

- This taught me a valuable lesson in dependency verification. In the world of open-source AI, a single typo can break a system. I now know to always verify model names and dependency configurations, a crucial skill for any forward-deployed engineer tasked with getting a system working on a client's infrastructure.

Navigating Distributed Systems 

- The final challenges were rooted in the complexities of connecting to a distributed system like Trino. I was plagued by a TrinoUserError stating Catalog 'hive' does not exist. My code was correctly making a request, but the underlying infrastructure was rejecting it. This was a classic configuration vs. code issue. My Python code was fine, but the configuration in my JSON file or my Trino server's setup was wrong.

The Milestone: 

- This was the most important lesson of all. It cemented my understanding that applied AI is as much about systems engineering as it is about machine learning. I had to debug not just my code but the entire stack: my local environment, the Docker container, and the Trino server. The fix was a two-part process: correcting the catalog name in my configuration and making my code resilient by handling potential None returns.

This open-source journey with Semantrino was more than just a coding exercise. It was a practical masterclass in the skills that define a forward-deployed engineer:

 - Systematic debugging of both code and infrastructure.

 - Adherence to framework conventions and data contracts.

 - Building resilient, modular systems that can withstand real-world failures.

This project validated my journey and served as a major milestone on my path to becoming a professional in the applied AI space. It showed me that getting an AI system to work in the real world is a collaborative effort between code, data, and infrastructure.
