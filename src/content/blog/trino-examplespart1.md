---
title: 'Trino + Python: Form Raw SQL to Clean JSON in 30 lines of code'
description: 'In this article, we developed a script for querying trino metadata'
pubDate: 'Jan 15, 2026'
category: 'Data Engineering and Data Infrastructure'
heroImage: '../../assets/images/2 - FA6Yb.jpg'
tags: ['ML']
---

Trino + Python: From Raw SQL to Clean JSON in  30 lines of code"


Data engineers and analysts spend a ton of their time just trying to figure out what data is available and where said data is. Before they can focus on building their pipeline, they have to understand the messy idiosyncrasies of their data environment. Today, we're using the Trino-Python client to build a script that you can run for discovery service that provides a data audit of your Trino cluster. We'll go through the code step by step to show how we built this and why we made certain architectural choices.


Logic Breakdown

First lets import our modules and craft our query

````
import trino
from trino.dbapi import connect
import json
query = "SELECT node_id,http_uri, state FROM system.runtime.nodes"
````



Configuring the Connnection



````
def configureConnection():
    conn = connect(
        host="localhost",
        port=8080,
        user="admin",
        catalog="tpch",
        schema="sf1"

    )
    return conn

````

So the first part of our script is where we do our configuration of the Trino connection. This is provided as part of the trino python client's python database API interface. We passed a connection string to the connect method containing our local trino environment credentials.


dataTransformation Helper

````
def transformQueryrowstoJson(rows,columns):
    data = []
    for row in rows:
        row_dict = {}
        for i, column in enumerate(columns):
            row_dict[column] = row[i]
        data.append(row_dict)
    return data

````

The second function that we use is our helper function that transforms the raw tuples that trino returns into raw tuples to save memory. Trino does the heavy lifitin of querying massive datasets but to save memory it tries to return data in a flat format. This manifests as tuples. This is inline with typical database drivers in the industry due to the memory efficiency. We'll "inflate" that data into dictionaries to provide a better experience for the developer using this pipeline for their infrastructure.

THe flow of this funciton is a bit complicated so lets imagine that we're mapping a spreadsheet. The columns are your headers, and rows are the data beneath them.

1. (data = []) We initialize an empty list to hold our finished products
2. (for row in rows) We step through and pick up one record at  a time. ex ('101,'active','coordinator') is one record
3. (row_dict = {}) For every row, we create a fresh dictionary to hold the key:value pairs.
4. (enumerate(columns)) This is where things get complicated. Enumerate allows us to iterate through the list tuple of columns and grab both the name and the position (index) of the column in the list
5. (row_dict[column] = row[i])We use the index i to grab the value from the row that matches the header name.
6. (data.append)Once the dictionary is full, we save it and append it to the list we initialized at the start of the function and move to the next row


Executing the Query

````
query = "SELECT node_id,http_uri, state, version FROM system.runtime.nodes"

def ExecuteQuery():
    conn = configureConnection
    cursor = conn
    try:
        cursor.execute(query)
        rows = cursor.fetchall()

        columns = [desc[0] for desc in cursor.description]
        return transformQueryrowstoJson(rows,columns)
    finally:
        cursor.close() 
````

This is where we put everything together and actually use the configureConnection method and the data transformation helper. First we create the connection, then we'll execute the query and fetch the rows. We then used list comprehension to parse out the descriptions for the headers. we passed this to our data transformation helper which returned our data in json format. We finished the funtion by using finally instead of except so we can close the connection.



Key Concepts

Let's go over some key concepts in our code that even advanced data engineers stumble over.
Enumerate() for data transformation

Lets step through using enumerate() because its equal parts a Pythonic superpower as well as a stumbling block for python developers. It solves the very specific problem of how do we get the value of an item and its position at the same time. 

Lets imagine that we're scanning a row of boxes. A for loop just looks at what is inside the box while enumerate looks at the label on the outise and the contents on the inside at the exact same time.


Now lets run our code to make sure it works
```
python3 ./Foundations/hello-world-pipeline.py
```

Here are our results
```
Executing Query
System Metadata:[{'node_id': '1841e3fcac99', 'http_uri': 'http://172.26.0.2:8080', 'state': 'active'}]
```



That's all we have for today! Next week we'll be working on developing a script for using the federated functionality of Trino. Thanks for reading and I'll see you in my next blog post.




