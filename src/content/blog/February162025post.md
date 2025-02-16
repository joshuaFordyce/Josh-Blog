---
title: 'SEP Deployment using Docker-Compose: Part 2'
description: 'In this article, we work on configuring connectors for Apache Iceberg, Minio, and Apache Hive to connect to Trino'
pubDate: 'February 25, 2025'
category: 'Data Engineering and Data Infrastructure'
heroImage: '../../assets/images/placeholder-hero.jpg'
tags: ['ML']
---

SEP Deployment using Docker-Compose: Part 2

Alright, so in the second part of this series, we’re going to jump into connecting our Trino cluster to some data sources. We’ll be using Docker to set up Postgresql, Apache Hive, and Apache Iceberg. Oh, and we’ll also be setting up MinIO for storage

Introduction to Apache Hive

So, Apache Hive is basically a data warehouse system that sits on top of Apache Hadoop. It allows you to use SQL to manage tons of data- Petabytes of data even! Hive is super fast at crunching through all that distributed data because it uses batch processing. It takes your HiveQL(a query language similar to SQL) and turns it into MapReduce or Tez jobs which then run on Hadoop. Finally, Hive keeps track of all of its metadata in something called a metastore, which essentially is a file-backed store that makes it easy to work with the data.

Introduction to MinIO

MinIo is an object storage system that works really well with data management systems like Apache Hive, Iceberg, and Postgresql. Object storage is a method of storing data where data is broken down into separate “objects” and kept in a flat structure. This is a lot different than the typical way of storing data in folders. Object storage is perfect for table formats like Iceberg and Hive because they use it to hold the data and use metadata to keep track of everything. This system makes querying and managing data in a data lakehouse or warehouse incredibly efficient.
  
Introduction to Apache Iceberg

Okay, so Iceberg is this open table format for analyzing data and the magic lies in its metadata files. These files are the actual brains behind Iceberg’s operation. The main metadata file keeps track of everything that is associated with the table. This includes the table’s schema, the data partitioning, special settings, and table snapshots. The Iceberg metadata is organized as a tree with the bottom holding the manifest files, the middle holding the manifest lists and the very top of the tree holding the main metadata file.

We’re going to zoom in on those manifest files first. They serve as a detailed logbook for our data which keeps tabs on the data files. This includes the important stats about each file as well as which files have been deleted. Cleverly, a single manifest file can even store stats for multiple Parquet data files. This means that you only need to open one file to work with the data which makes reading metadata way faster.

Next, we have the manifest list. It tracks the state of the data in a table snapshot. Iceberg uses this list to figure out which data files to read or write to when you’re doing data warehouse stuff, so it is key to effective data management

Now we have the Final Boss, the metadata.json file! This is the central control center for all the table metadata and it is responsible for a few crucial things:

  - It helps optimize storage and queries by defining how the data is partitioned
  - It's the single source of truth for all the table's metadata, making it easy for the data engines to understand the data
  - It records snapshots, which lets you do “time travel” queries - meaning you can query the table as it was at some point in the past 
So now that we’ve covered that, the metadata.json file is a big deal in Iceberg. If you want to dive deeper into the metadata.json file, there is a blog article here that explains it in more detail.

Introduction to Postgresql

Okay, for this article we’re going to keep our explanation of Postgres quite simple. It’s an open-source relational database, and in our case, we’re basically just using it to store the Starburst insights metrics. That’s all we’ll say about Postgres for now- if you’d like a deeper exploration of Postgres, check out this Medium article. 
	
Hands-on Configuration

Now that we have introduced the components of our infrastructure, let's get our hands dirty. The first step that we need to take is to navigate to our directory where we have our “compose.yaml”. Currently, we only have the configuration for a Starburst container to be running. Let's change that by adding configuration code for Hive, minio, Postgres, and Iceberg.

Here is an example of the configuration code that we need to add to the compose.yaml file that we first created in the first part of our blog series.

```
hive:
   container_name: hive-metastore
   hostname: hive-metastore
   image: 'ghcr.io/trinodb/testing/hdp3.1-hive:latest'
   ports:
     - 9083:9083
   environment:
     HIVE_METASTORE_DRIVER: org.postgresql.Driver
     HIVE_METASTORE_JDBC_URL: jdbc:postgresql://postgresql:5432/sep?ssl=false
     HIVE_METASTORE_USER: hive
     HIVE_METASTORE_PASSWORD: hive
     HIVE_METASTORE_WAREHOUSE_DIR: s3://minio/jdizzy/
     S3_ENDPOINT: http://minio:9001
     S3_ACCESS_KEY: qW9v9wHXIukrixsznJUc
     S3_PATH_STYLE_ACCESS: "true"
    
     S3_SECRET_KEY: V36JqoqLEcocqul6Kvgs0XZQpiYomCqn3wjp5yki
   depends_on:
     - postgres
  


 minio:
   hostname: minio
   image: minio/minio:latest
   container_name: minio
   ports:
     - 9000:9000
     - 9001:9001     
   volumes:
     - ./minio/:/data
   environment:
     - MINIO_ROOT_USER=admin
     - MINIO_ROOT_PASSWORD=trinoRocks15
     - MINIO_ADDRESS=:9000
     - MINIO_CONSOLE_ADDRESS=:9001
     - MINIO_ACCESS_KEY=qW9v9wHXIukrixsznJUc
     - MINIO_SECRET_KEY=V36JqoqLEcocqul
   command: minio server /data


 postgres:
   image: postgres:11
   container_name: "postgresql"
   user: "postgres"
   hostname: "postgres"
   ports:
     - 5432:5432
   environment:
     POSTGRES_USER: admin
     POSTGRES_PASSWORD: trinoRocks15
     POSTGRES_DB: sep
   
 starburst:
   # image: starburstdata/starburst-enterprise:429-e.0
   # image: starburstdata/starburst-enterprise:435-e.9
   image: starburstdata/starburst-enterprise:453-e
   # image: starburstdata/starburst-enterprise:438-e
   # image: starburstdata/starburst-enterprise:433-e
   container_name: starburst
   ports:
     - 8080:8080
   volumes:
     - ./coordinator.config.properties:/etc/starburst/config.properties
     - ./starburstdata.license:/etc/starburst/starburstdata.license 
     - ./backend_svc.properties:/etc/starburst/backend_svc.properties
     - ./postgresql.properties:/etc/starburst/catalog/postgresql.properties
     - ./hive.properties:/etc/starburst/catalog/hive.properties
     - ./iceberg.properties:/etc/starburst/catalog/iceberg.properties
     - ./password-authenticator.properties:/etc/passowrd-authenticator.properties
     - ./password.db:/etc/password.db




```
Next, we need to add the following configuration information to our config.properties. 
```
http-server.https.enabled=true
http-server.https.port=8443
http-server.https.keystore.path=etc/certificate.pem
http-server.authentication.type=PASSWORD
starburst.access-control.enabled=true
```



Next, we can create catalog files for our connectors so Starburst knows how we want them configured.

Hive.properties example
```
connector.name=hive
hive.metastore.uri=thrift://hive-metastore:9083
hive.metastore.username=
hive.s3.endpoint=http://minio:9000
#hive.s3-access-key="qW9v9wHXIukrixsznJUc"
#hive.s3-secret-key="V36JqoqLEcocqul6Kvgs0XZQpiYomCqn3wjp5yki"
hive.s3.path-style-access=true
#hive.allow-drop-table=true
#hive.config.resources=/hive-site.xml
fs.hadoop.enabled = true
#hive.metastore.username = admin
```



Iceberg.properties
```
connector.name=iceberg
hive.metastore.uri=thrift://hive-metastore:9083
hive.s3.endpoint=http://minio:9000
fs.hadoop.enabled = true
hive.metastore.username = admin
```


Postgresql.properties
```
connector.name=postgresql
connection-url=jdbc:postgresql://postgres:5432/sep?ssl=false
connection-user:admin
connection-password=trinoRocks15
```



Alright, with everything configured, we should be able to run docker-compose up to spin up our four containers

Now that we have this configured we should be able to run the “docker-compose up” to run the four containers that are required for this infrastructure.

You should be able to view the Starburst Insights page and run a sample query on the Iceberg catalog. You can test this by running a basic Select * from Iceberg.schema.table command.

In this article, we’ve worked on building a very basic version of a containerized Starburst deployment with some very typical connectors for Trino to run queries across. We attached the Hive connector and the Iceberg connector. Then we configured a Postgresql connector to work as our metadata store for the Starburst insights page. The next article in this series will focus on benchmarking the query performance of the Iceberg, Delta Lake, Avro, ORC, and Parquet table formats when running an ETL pipeline.

Thanks for reading and I’ll see you in the next article!

