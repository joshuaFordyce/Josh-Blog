---
title: 'Asynchronous SQL Gateway: Concurrency and the API Contract'
description: 'In this article, we explored concurrency and the API Contract in the Semantrino API'
pubDate: 'November 17, 2025'
category: 'Data Engineering and Data Infrastructure'
heroImage: '../../assets/images/pexels-picjumbo-com-55570-196646.jpg'
tags: ['ML']
---
The Asynchronous SQL Gateway: Concurrency and the API Contract

Async vs. Blocking: The MLOps Rationale

When I started designing the Semantrino API I wanted to maintain high throughput for an I/O-bound service. To do this, I built it using the FastAPI framework which is asynchrnous. So when our service makes external calls the time is spent in network latency. This is perfect when fetching a large query plan from Trino or waiting ror the LLM API response. If we used syncrhnous code, Pythons GIL to stall the entire process which would cripple our application. By using async and await, we were able to relase the GIL. THis allowed us to use a single event loop to potentially manage thousands of concurrent requests efficiently. 

API Contract(Pydantic Validation)

The API contract is the non-negotiable definition of the system's inpyuts and outputs. To establish this contract, we choose Pydantic. We defined a QueryRequest Pydantic model that receeives the user's input and then FastAPI validates this request at the network edge. This ensures that any malformed request is immediately rejected which helps us prevent our application from wasting resources on invalid inputs.

Architectural Philosophy:

The endpoint handler acts purely as a translator. The FastAPI route's job is simpply to receive HTTP< validate the Pydantic schema and then delegate the relevant complex work to internal service classes. This ensures that we continue to decouple the business logic from the api framework.


Conclusion:

In this first part of the series on the semantrino API, We've went over establsighing the foundationfor an intelligent data agent. We've focused on embracing the asynchronous model of Python and FAstapi, using the Pydantic QueryRequest Model as our primary API contract and security guardrail and ensuring that we decouple the service layer from the business logic. In part 2, we'll dive deed into how we used the LangChain framework , the Singleton Trade-Off and the context window.
