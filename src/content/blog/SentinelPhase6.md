---
title: 'The Art of Passing JSON with jq: From Messy Data to Actionable Insights'
description: 'In this article, I described how to use the jq cli tool to parse the raw JSON we received from the '
pubDate: 'December 10, 2025'
category: 'DevOps and Cloud Infrastructure'
heroImage: '../../assets/images/ventura.jpg'
tags: ['ML']
---


The Art of Parsing JSON with jq: From Messy Data to Actionable Insights

Making a direct API call to the Kubernetes Metrics Server is a powerful technique. However, the data returned is not a clean, human-readable table. Instead, it's a raw, structured JSON response—a format designed for machines, not for shell scripts. This presented our next challenge: how do we parse this messy data to extract the specific information we need?


The solution is a powerful and elegant command-line tool called jq. It's often referred to as a "sed for JSON data," and it's an essential tool for any modern DevOps practitioner.


The Problem with Raw JSON:

The Metrics Server response is a JSON object with nested keys and arrays, like items, metadata, and usage. Without a dedicated tool, parsing this in a shell script would require a complex series of grep and awk commands, which would be brittle and prone to failure if the JSON structure ever changed slightly.
jq provides a simple, expressive syntax for traversing and filtering JSON.


Our jq Command in Action
To get the metrics for our specific container, we crafted a jq filter that did the following:
items[]: This iterates over the array of pods returned in the items key.
select(.metadata.name=="$1"): This is a filter that finds the one specific pod whose metadata.name matches the argument we provided to our script.


.containers[]: This iterates over the list of containers within that specific pod.
{pod: .metadata.name, ...}: This creates a new, simplified JSON object containing only the fields we care about: the pod name, container name, CPU usage, and memory usage.


This single, elegant jq command replaces what would have been dozens of lines of brittle shell scripting. It transforms the raw, complex JSON into a simple, actionable format that we can easily use in our script to print a clean report.


The lesson here is that while the shell is powerful, it has its limits. For specialized tasks like JSON parsing, using the right tool for the job—in this case, jq—makes the script more robust, readable, and maintainable. It turns a seemingly impossible task into a simple, three-line solution.
