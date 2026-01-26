---
title: 'Gateway API for Trino Part 4 '
description: 'In this article, we explored hidden latency taxes in Kubernetes based Trino'
pubDate: 'Jan 26, 2026'
category: 'Data Engineering and Data Infrastructure'
heroImage: '../../assets/images/2 - FA6Yb.jpg'
tags: ['ML']
---

Gateway API for Trino Part 4: Beyond the Averages

Introduction
For the fourth article in this series, I want to step through a primer on latency that is specific to distributed query engines running on Kubernetes. We are going to look at performance and how to measure latency in a way that actually reflects the health of your cluster.
We want to focus on tail latency because this represents the response time of your slowest requests. Data engineers are often fooled by "good" average latency numbers (P50), thinking the user experience is fine. The issue is that averages hide the outliers; the true user experience for complex workloads is found at the high percentiles: P95, P99, and P99.9.

Quick Rundown on Percentile Mapping
Latency percentiles map the distribution of response times to measure performance consistency. They indicate the maximum latency experienced by a specific percentage of users:
P50: Represents the typical experience; 50% of queries are faster, and 50% are slower.
P90/P95 (Tail Latency): Reflects the experience of users with complex queries or during periods of high concurrency.
P99: Represents the slowest 1% of queries. This is the "danger zone" that highlights severe performance issues like CPU throttling or memory pressure.

Trino-Specific Latency Traps

Scatter-Gather Amplification
Since we’re dealing with a distributed SQL engine, we have to talk about Scatter-Gather Amplification. This is the fundamental pattern for parallel execution: when you submit a query, the Coordinator "scatters" the work by breaking it into splits and assigning them to dozens or hundreds of workers. Once the workers process their local data, the Coordinator "gathers" those results for the final aggregation or join.
The core issue is that the gather phase is synchronous—it cannot complete until the last worker finishes its task. If you have a cluster of 100 worker nodes and each has a 99% probability of responding within 100ms, you might think the query will take 100ms. However, the math of probability tells a different story:
$$P(\text{at least one delay}) = 1 - (0.99)^{100} = 63.4\%$$
Even if 99 workers are running perfectly, there is a 63% chance your query will be delayed because a single straggler node is taking its time. The more we parallelize and add workers, the more vulnerable we become to these outlier nodes.

East-West Latency
East-West traffic refers to the network traffic moving between the Coordinator and the Workers, or between the Workers themselves. In Trino, there are three distinct types:
Heartbeats: Workers constantly send status updates to the Coordinator.
Task Assignment: The Coordinator sends task descriptions and instructions to the Workers.
Data Exchange (The Heavy Lifter): Workers re-partition data and send it to other workers. This is most common during Joins and GROUP BY operations.
The "flow" of data exchange is actually a pull-based system managed through a series of buffers. First, an Upstream Worker finishes a task, stores the result in its RAM (Output Buffer), and notifies the Coordinator. Second, a Consuming Worker is assigned the next stage; it looks at the metadata and sends an HTTP request to the Producer saying, "Give me the data for Partition X." Finally, the Producer streams the data to the Consumer, who places it in an Exchange Client Buffer and acknowledges receipt. High latency at any point in this "pull" creates a backpressure ripple effect that slows the whole cluster.

North-South Latency
While East-West is how the cluster "thinks," North-South latency is how fast the cluster can "talk" to the outside world. This is the time it takes for a request to travel from an external user or app to the Trino Coordinator and for the results to travel back out.
The workflow is straightforward: The Client sends the SQL statement $\rightarrow$ The Coordinator parses it and creates a plan $\rightarrow$ Trino produces rows and buffers them on the Coordinator $\rightarrow$ The Coordinator sends that data to the Gateway/Ingress $\rightarrow$ The Gateway sends the data to the User.

Kubernetes Layer Latency Tax

Encapsulation and Decapsulation
These are the "wrapping" and "unwrapping" processes that happen every time a data packet moves across the network. In Trino environments, this "CNI tax" is high because of the massive volume of data moved during shuffles.
Encapsulation: The packet starts at the App layer and moves down. The Transport Layer adds a TCP header, the Network Layer adds an IP header, and the Data Link Layer adds an Ethernet frame.
Decapsulation: This process is reversed at the destination. Each layer peels off its respective header until only the original Trino data remains.
Most CNIs (like Calico) use an Overlay Network, which adds a second layer of encapsulation. The original IP packet is wrapped inside a VXLAN or UDP packet to navigate the physical network. This "outer wrapper" adds roughly 50 to 70 bytes of extra data to every packet, eating into your bandwidth and increasing CPU overhead.

CFS Tax
While encapsulation is a network tax, the CFS Tax (Completely Fair Scheduler) is a timing tax. This is the Linux Kernel’s implementation for enforcing CPU limits.
If you set a limit of 2.0 CPUs, Kubernetes gives your container 200ms of CPU time for every 100ms window. Because Trino is highly concurrent, a worker might have 10 active threads. If each thread works for 20ms, you consume your entire 200ms quota in just 20ms of real time. For the remaining 80ms of that window, the Linux Kernel stops the container, throttling the worker until the next cycle begins. This happens even if the physical host has plenty of idle CPU, creating those P99 spikes we’re trying to avoid.
We’ve now pulled back the curtain on the latency of the Kubernetes layer. Understanding these latency traps is important to our experiment pitting the Gateway API vs Ingress object. Now that we’ve explored the mechanics of how the latency in how the Cluster talks and thinks, we’ll examine the hard data. In  the next article, we will examine the results of the experiment we set up in the previous installment to see which architecture actually survives the P99 stress test.. Thanks for reading and I’ll see you in the next article!

Conclusion


