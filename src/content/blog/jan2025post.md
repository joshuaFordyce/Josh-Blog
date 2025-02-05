---
title: 'SEP Deployment using Docker-Compose: Part 1'
description: 'In this article, we work on configuring a Containerized version of Starburst Enterprise using Docker-Compose and Docker '
pubDate: 'January 06 2025'
category: 'Data Engineering and Data Infrastructure'
heroImage: '../../assets/images/placeholder-hero.jpg'
tags: ['ML']
---


SEP Deployment using Docker-Compose: Part 1 
Pre Requisites for this Article:

While this tutorial is intended for beginner to intermediate technical audiences there are some prerequisites. We will need docker and docker desktop installed on your machine. Once we get that done we need to chat about docker containers and docker compose.yaml scripts. 

Containerization:
Containerization is a process consisting of packaging your code and its dependencies into a logically isolated box. While there are different types of containers, docker containers are by far the most popular so we’ll focus on docker. The way docker works is by using three fundamental steps. The first step is building what's called a Docker file. A docker file is a logical blueprint with instructions on how to package up the code and dependencies needed to run your application. The next step is building the docker image with the dockerfile and you typically do that with the command “Docker build -t app”. The third step is running the docker image which turns it into an actual containerized version of your application running in a logically isolated box on your host computer.

Here is an example:
```
# syntax=docker/dockerfile:1

FROM node:18-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
EXPOSE 3000
```



Docker Compose:
	Docker-Compose is a tool offered by Docker to allow applications to run using multiple containers. With Docker-Compose, You're able to manage several different infrastructure components such as networks, volumes, and services in a single yaml config file. There are four key bash terminal commands for working with Dockerfiles. The first command is a“docker-compose up” to start the services defined in your dockerfile. The second command is  “docker-compose down”  which we use to stop the services once they're already running. The third command is “docker-compose logs <target-container-name> ” and it prints the logs of the target container. The fourth command is “docker-compose ps” and it allows us to print the services that you have running.

Here is an example:
```
services:
  frontend:
    image: example/webapp
    ports:
      - "443:8043"
    networks:
      - front-tier
      - back-tier
    configs:
      - httpd-config
    secrets:
      - server-certificate

  backend:
    image: example/database
    volumes:
      - db-data:/etc/data
    networks:
      - back-tier

volumes:
  db-data:
    driver: flocker
    driver_opts:
      size: "10GiB"

configs:
  httpd-config:
    external: true

secrets:
  server-certificate:
    external: true

networks:
  # The presence of these objects is sufficient to define them
  front-tier: {}
  back-tier: {}

```
This YAML config will spin up an application with 1 application service, 1 database service, 2 networks, 1 secret, and 1 persistence volume.

Deploying SEP with Docker Compose
Now that we have a general understanding of Docker and Docker-Compose, we can continue with our deployment of Starburst Enterprise. Let's have a quick chat about the architecture of a general SEP deployment

As we said before, trino is a distributed SQL query engine. The Trino cluster has two different node types in a client-server architecture style. The coordinator node is the control server that processes queries from the outside client, parses and analyzes the queries, and performs the scheduling of tasks among worker nodes. The worker nodes are the servers that carry out the actual tasks of accessing data from the different data sources connected to the cluster and then processing this data. To scale Starburst enterprise horizontally you can add more workernode servers. You can also scale vertically by increasing the size of the worker nodes. To connect data sources to Trino, we use configurations called catalogs to configure connectors. 

Now that we’ve gone through our basic prerequisites we can start building our SEP deployment. The first thing we need to do is create a new directory for our SEP deployment by running the command "mkdir starburst". Next, we need to navigate to our newly made directory using the command "cd starburst". Next, we need to generate a Starburst data license by requesting a provisional license from Starburst. Once we’ve generated our Starburst data license, then we can create a config.properties file by running the command "vi config.properties".
Here is an example of the config.properties file we need to create
```
http-server.http.port=8080
discovery.uri=http://localhost:8080
insights.jdbc.url=jdbc:postgresql://postgres:5432/sep
insights.jdbc.user=admin
insights.jdbc.password=trinoRocks15
insights.persistence-enabled=true
insights.metrics-persistece-enabled=true" >~/Downloads/starburst/config.properties"

```
This config properties file is what tells our Starburst container to run the application on port 8080. The config properties also tell our SEP deployment where to launch the insights tool. The Starburst insight tool allows us to export our cluster metrics and store the cluster metrics in our Postgresql database running on port 5432.

Next, we want to configure a postgresql.properties for the postgresql catalog. We can create a postgresql.properties file by running the command “vi postgresql.properties”.
Here is an example of the postgresql.properties file we have to create.
```
connector.name=postgresql
connection-url=jdbc:postgresql://postgres:5432/sep?ssl=false
connection-user:admin
connection-password=trinoRocks15
```



Now that we have our config file, our Starburst data license, and our catalog file created we can start working on our compose file. We can create compose.yaml by running “vi compose.yaml”.
Here is an example of compose.yaml file we have to create.
```
services:
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
   image: starburstdata/starburst-enterprise:443-e.7
   # image: starburstdata/starburst-enterprise:438-e
   # image: starburstdata/starburst-enterprise:433-e
   container_name: starburst
   ports:
     - 8080:8080
   volumes:
     - ./coordinator.config.properties:/etc/starburst/config.properties
     - ./starburstdata.license:/etc/starburst/starburstdata.license 
     - ./postgresql.properties:/etc/starburst/catalog/postgresql.properties
```




Now that we have this working, you should be able to run “docker-compose up—d,” which should get your application up and running. You should also be able to navigate to the port Starburst is running on. If you have any issues getting your code to run, please post questions in the comments or refer to the GitHub repository using the link below.
https://github.com/joshuaFordyce/DockerTrinoTutorial
