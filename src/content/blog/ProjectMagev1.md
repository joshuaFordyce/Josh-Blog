---
title: 'The complexity of Data Lakehouse Deployments on Kubernetes '
description: 'In this article, we explored the concept of Partitioning in Data Warehouses'
pubDate: 'August 29, 2025'
category: 'Data Engineering and Data Infrastructure'
heroImage: '../../assets/images/2 - FA6Yb.jpg'
tags: ['ML']
---





The hidden complexity of Data Lakehouse Deployments on Kubernetes:

Working with open source Trino can be a difficult task for customers that want to stay on open source and develop tooling using an open source data stack. Lets say a data engineer want’s to set up a data lakehouse made up of all open source tools. Lets imagine a setup where they’d use Trino as their query engine layer, postgres/Mongo/Oracle/Minio as their data sources, and hive as the metastore that stores their metadata. All of these are open source tools that come together to make a complex data stack that forms the foundation of a data infrastructure for important data reliant open source projects.

Lets walk through the process of managing these tools on a platform like kubernetes. Kubernetes tends to be the most popular way to deploy complex data infrastructure like the underpinnings of a data lakehouse for a variety of reasons. 

Pros of Deploying to Kubernetes:
Portability
A data lakehouse deployed on kubernetes can run anywhere without a lot of changes to the application code which allows for developers to prevent vendor lock-in and gives a single consistent deployment and operational model
Scalability
K8’s built-in orchestration tools allow you to auto scale your Trino workes up and down based on demand
Resilience
If a Trino Pod or a Mino instance fails, the cluster will automatically detect the failure and restart the pod which ensures a self-healing infrastructure

While those pros are great, we’re here to talk about the drawbacks. 

Drawbacks of Lakehouse deployment to Kubernetes:
Operation Complexity
Manually writing and managing tons ofYAML manifests for a full Trino deployment, much less a full Data lakehouse deployment is a massive headache for a developer. 
Inconsistent Environments
With manual deployments, it's difficult to guarantee that a development cluster is a perfect clone of a production cluster which can then lead to hard to reproduce bugs.
Lack of Lifecycle Management
When a developer manually deletes a Trino cluster or Minio instance, they might forget to delta a hanging service or configmap which can lead to orphaned resources. This can clutter the Kubernetes cluster and drive up costs.


The TrinoOperator is a Kubernetes-native solution that I built that solves these drawbacks while preserving the pros of a Kubernetes based Lakehouse deployment. This tool allows a single open source developer to deploy a production ready lakehouse with a single command. The idea is that we want to free up valuable engineering time for them to focus on the products they’re trying to build instead of the data ops that underpin those projects. The operator acts as a reliable layer of abstraction, ensuring that the open-source data stack is always consistent, self-healing and easy to manage

Lets talk about how the TrinoOperator  to solve the drawbacks of Lakehouse deployment to Kubernetes

The Trino Operator solves the operation c by acting a single source of truth and the TrinoCluster custom resources functions as the entire deployment as code. 

Solves for the drawbacks:

Operation Complexity
The TrinoOperator solves this by providing a unified, declarative API. Basically, the developer defines their desired ata lakehouse stack ina single TrinoCluster custom resource and then the operator handles all of the underlying complexity
Ensuring Consistent environments
The TrinOperator solves this by acting as a single source truth. This means that the custom resource is the entire deployment as code and by changing a few variables in this single manifest you can deploy a development, staging or prod cluster
Managing an Unreliable Lifecycle
The TrinoOperator solves this issue by implementing the Finaliezer pattern. This Finalizer will act as a custom garbage collector in that when the user deletes the custom resource, it will delete all of the associated resources to prevent orphaned resources.


In the next article, we’ll take a deep dive into the first iteration of the Project Mage :TrinoOperator. We’ll walk through the development of the project and the troubleshooting that we went through to test the project.
