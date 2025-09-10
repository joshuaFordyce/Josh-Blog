---
title: 'The Architecture of the TrinoOperator'
description: 'In this article, we explored the architecture of the TrinoOperator'
pubDate: 'August 28, 2025'
category: 'Data Engineering and Data Infrastructure'
heroImage: '../../assets/images/a.jpg'
tags: ['ML']
---




As I said in part 1, the TrinoOperator is a Kubernetes native solution that is designed to automate the management of a Trino Cluster. I wanted to provide a high level overview of the operator’s architecture and how I implemented the reconciliation loop.

Architecture:

The Operator’s architecture is built using the fundamental principle of a control loop. The control loop is composed of two different layers. THe first is the Custom resource. This is the declarative blueprint that functions as a single, human-readable YAML file that defines the entire desired state of the trino deployment. This handles things from the number of the workers, to the config of the catalogs. Then we have the Go Operator. This functions as the “watcher” of the system that that ensures that the actual state of the cluster will always match the desired state in the CR.

Overview of Reconciliation Loop

The heart of the operator is the reconciliation loop. This loops sis designed to be idempotent which allows it to be run multiple times but it wil only actually make changes if the actual state differs from the desired state. 

Behavior of the Reconciliation Loop
1.The reconciliation loop starts by grabbing the TrinoCluster custom resource. This fetches the user’s desired config and sets up as the blueprint for all subsequent steps
2. The Operator’s function then will call a series of helper function to reconcile the core infra components.
	Heres how the operator performs its job
First it reconciles a deployment for the Trino coordinator and another for the workers. This ensures the number of replicas and the pod specs match the CR
The operator reconciles a service to then provide a stable network address for the coordinator
Then we reconcile a configmap to provide the config files for the trino pods

3. Update the Status: Now that the infra has been reconciled, the operator can focus on the observed state of the cluster and then sends it back to the status field of the trinocluster cr. This allows the developer to access a real-time, auditable report of the cluster’s health

4. The operator then will finally focus on implementing a finalizer pattern. This ensures that when a user deletes the trinocluster custom resource that the operator will make sure it cleans up all of the associated resources which allows for us to prevent orphaned resources from being left scattered around the cluster. 

In our next article, we’ll talk about the automated catalog manage feature that we’ve implemented in the operator. I'm extremely excited about it since it allows the user to leave behind the manual, error-prone process of configuring catalogs for a Trino cluster.
