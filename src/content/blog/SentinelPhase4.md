---
title: 'Phase One of the Database Sentinel: Part 2'
description: 'In this article, I continue describing how I went about implementing the first part of the Database Sentinel'
pubDate: 'November 20, 2025'
category: 'DevOps and Cloud Infrastructure'
heroImage: '../../assets/images/green-leaves-close-up-portrait.jpg'
tags: ['ML']
---

From Dependency Hell to Docker Success: Part 2

Continuing our Troubleshooting journey, we have more issues. We had key issues with installing Cassandra and Postgres and we've detailed it in the article.
The Cassandra Conundrum: 

- Our first challenge was getting Cassandra to start. The logs showed a series of Unrecognized VM option errors, starting with -XX:+UseConcMarkSweepGC. This option is for an old garbage collector that was removed from modern versions of the Java Virtual Machine (JVM).


Principle 1: 

- The error message is your first clue. We knew the problem was related to JVM configuration. Our initial thought was to fix a single file, jvm11-server.options, by using a sed command in our Dockerfile to comment out the bad option and enable the correct one (-XX:+UseG1GC).
The error persisted. This revealed a deeper issue: the outdated option was being injected by a different startup script, likely cassandra-env.sh, which has conditional logic to set JVM options. The fix required a more iterative and systematic approach. Each time a new, related error (CMSParallelRemarkEnabled, CMSInitiatingOccupancyFraction, etc.) appeared, we added another sed command to our Dockerfile. We were effectively removing a whole block of outdated configuration options until the JVM had no choice but to start correctly.

- The takeaway here is that a clean configuration file doesn't guarantee a clean startup. Understanding the chain of command—from the Dockerfile to the shell scripts to the configuration files—is crucial.

The PostgreSQL Puzzle: 


- Next, we tackled a completely separate issue: pg_ctl, the PostgreSQL control utility, was not running. Our first error was bash: pg_ctl: command not found.


Principle 2: 

- Start with the environment, not the application. Before assuming the application is broken, check the basics. A command not found error almost always points to an issue with the PATH environment variable. Our investigation revealed that the directory containing pg_ctl was not in the PATH , and a simple export command fixed it.


This fix led to our next error: pg_ctl: 

- cannot be run as root. This is a built-in security measure in PostgreSQL. It correctly revealed a permissions issue. We then successfully used su - postgres to switch to the correct, unprivileged user.


Principle 3: 
- A successful fix can reveal the next problem. The user switch, however, led back to the original error: pg_ctl: command not found. This demonstrated a key point about shell environments in Linux:
    - the root user and the postgres user have different PATH environments. The final solution was to again use an export command, this time within the postgres user's shell session, to add the PostgreSQL binary directory to its PATH.


Conclusion:
A Methodical Approach to Debugging

Debugging multi-service container environments is a process of discovery. It’s rarely a single-step fix. These examples show that the key to success is a methodical, step-by-step approach.
- Read the error messages carefully. They are your first and best clues.
- Verify the obvious things first. Don't assume. Is the file really in the correct location? Is the PATH set correctly?
- Understand the entire execution chain. Don't just look at the application's configuration file; look at the startup scripts and environment variables that are being set along the way.

Be prepared for iterative fixes. Fixing one issue often reveals another, and a stubborn problem may require a series of small, targeted corrections to solve.
