---
title: 'Phase One of the Database Sentinel'
description: 'In this article, I describe how I went about implementing the first part of the Database Sentinel'
pubDate: 'November 10, 2025'
category: 'DevOps and Cloud Infrastructure'
heroImage: '../../assets/images/book.jpg'
tags: ['ML']
---

This article focuses on the overall Phase 1 of Sentinel. We'll talk about mastering the fundamentals of database reliability by builiding a portable test enviornment and performing essential manual diagnositics.
Our goal is to establish a core knowledge base of bare-metal and system-level checks, which are the building blocks for our future automation.


Before we can monitor a database, we need to have one to moinitor. In this phase we're not just going to set up one but a diverse nvironment with PostgreSQL, MYSQL, REDIS, and Cassandra. 
Instead of manual installations on a vm, it's easier for  Dockerfile to create a reproducible, portable and low-overhead environment.
This Dockerfile will orchestrate the setup. It will start from a base Linux Image and then run a series of commands to install each database and its dependencies.
It will also expose the necessary ports for each service and include a startup script to ensure all services are ready.
This containerized approach means we can quickly spin up and tear down our test environment

Once the docker container is running the real work begins. We performed manual checks using bare-metal and systemd tools. This hands-on experience is great for understanding what the automated scripts are doing under the hood

We start by doing Bare Metal Resource Checks that look at the machine's vitals. We use simple Linux command-line tools to diagnose resource consumption. 
For the CPU, commands like top or htop give us a real-time view of CPU and memory usage which helps us identify runaway processes or high load.
For the Memory, free -m provides a quick snapshot of available memory. We then used the iostat and df-h to understand the disk read/write activity and disk space usage. Monitoring these prevents a database from failing due to a full isk or an I/O bottleneck

On modern Linux systems, Systemd is the service manager. We use it to check if our database services are running as expected. Systemctl Status provides a detailed output including if the service is active, its process ID and the latest log entries
The systemctl is-active is perfect for scripting as it returns a simple exit code that the scripts can act on.
The final part of this first manual phase is testing connectivity & Port Listening. We use commands like ss -ltn or netstat -ltn to verify that each database is listening on its designated port.

Once we worked through the manual testing phase, we moved on to transforming our knowledge into a reusable, packagable, and portable tool. 
We created a script that will take the manual checks we've learned and automate them. It should systematically go through each database, check its systemd status, run basic bare-metal checks for CPU,memory, and disk space. 
While simple, this script represnts the core philosophy of the project: creating specialized, low-overhead tools that are transparent and easy to understand. In the next phases, we'll explain how we expanded on this foundation.
