---
title: 'SEP Deployment using Docker-Compose: Part 2'
description: 'In this article, we work on configuring connectors for Apache Iceberg, Minio, and Apache Hive to connect to Trino'
pubDate: 'February 25, 2025'
category: 'Data Engineering and Data Infrastructure'
heroImage: '../../assets/images/placeholder-hero.jpg'
tags: ['ML']
---

Making Iceberg Tables Faster: A Friendly Guide to Statistics
Hey everyone! Recently, I've been helping folks optimize their Apache Iceberg tables with Trino, and I noticed something interesting: table statistics were causing a bit of confusion. Even for me! After some digging, I realized there wasn't a really clear, practical guide out there. So, I thought, why not write one? Let's demystify Iceberg table statistics together!
Why Bother with Table Statistics?
Before we dive into the "how," let's chat about the "why." You see, when Trino (or any SQL engine) runs a query, it follows a specific order of operations. Think of it like a recipe:
Find the Ingredients (FROM/JOIN): First, it figures out which tables you're using and how they connect.
Filter the Ingredients (WHERE): Then, it picks out the rows that match your criteria.
Group the Ingredients (GROUP BY): Next, it organizes the rows into groups.
Filter the Groups (HAVING): It filters those groups based on your conditions.
Pick What You Want (SELECT): It chooses the columns you asked for.
Put Them in Order (ORDER BY): It sorts the results.
Take a Slice (LIMIT/OFFSET): Finally, it gives you just the number of rows you need.
Knowing this order is super important because it helps us understand how Trino decides the best way to run your query. That's where the Cost-Based Query Optimizer (CBO) comes in.
The CBO: Your Query's Personal Chef
Imagine the CBO as a chef in a busy kitchen. It has many ways to cook your meal (your query), but it wants to choose the fastest and most efficient one. To do that, it needs information about the ingredients (your data). That's where table statistics come in!
Think of table statistics as a detailed ingredient list. They tell the CBO things like:
How many rows are in the table?
What are the minimum and maximum values in a column?
How many unique values are there?
How many null values are there?
How big is the data?
With this information, the CBO can make smart decisions, like choosing the best indexes and join orders, to get your query results faster. Basically, statistics help Trino make the best choices so that you get your data faster.
Let's Get Practical: Working with Iceberg Statistics
Okay, enough theory! Let's get our hands dirty with some Iceberg commands.
1. Gathering Statistics: The ANALYZE Command
Imagine you want to get a general overview of your Iceberg table. The first thing you'll do is run the ANALYZE command:
SQL
ANALYZE table_name;
This command gathers some essential statistics like the data size and the number of distinct values. It also calculates some quick stats on the fly:
Minimum value
Maximum value
Value count
Null value count
Now, you might not need statistics for every single column. Usually, you'll focus on columns used for filtering, grouping, or joining. You can specify which columns to analyze like this:
SQL
ANALYZE table_name WITH (columns = ARRAY['col_1', 'col_2']);
2. Keeping Statistics Fresh: Running ANALYZE Again
As your data changes, your statistics can become outdated. To refresh them, just run the ANALYZE command again! It's that simple.
3. Removing Statistics: DROP_EXTENDED_STATISTICS
If you ever need to remove the extended table statistics, you can use the DROP_EXTENDED_STATISTICS command.
Wrapping Up
I hope this article has helped you understand Iceberg table statistics a little better. They're a powerful tool for optimizing your Trino queries, and with a little practice, you'll be a pro in no time!
Thanks for reading, and I'll catch you in the next one!
