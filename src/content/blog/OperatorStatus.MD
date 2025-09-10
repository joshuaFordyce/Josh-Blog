---
title: 'The Operator's Report Card'
description: 'Discussing the communication channel between the operator and user'
pubDate: 'October 15, 2025'
category: 'DevOps and Cloud Infrastructure'
heroImage: '../../assets/images/bg.jpg'
tags: ['ML']
---

The Operator's Report Card: Implementing Status Updates 
The Status field is a communication channel between your operator and the user. It's the difference between a kubectl get command returning a blank status and a rich, informative one that tells the user the health and state of the cluster. A production-ready operator continuously updates the status of its custom resource to reflect the actual state of the cluster.

1. The Core Concepts: Spec vs. Status 
The core of this concept is the distinction between a resource's desired state and its observed state.
Spec (The Desired State): This is the blueprint for your cluster. It's what the user defines in the TrinoCluster manifest. It contains all the fields that tell your operator what to do.
Status (The Observed State): This is the report card. It's what your operator sees in the cluster after it's finished reconciling. It tells the user if the blueprint has been successfully implemented.
A robust operator continuously updates the Status field of its custom resource to reflect the actual state of the cluster. This is the foundation of a self-healing system.

2. The Implementation: A Step-by-Step Guide 🛠️
Step 1: The Blueprint for the Status Field
First, you must add a Status field to your custom resource's definition in trinocluster_types.go. This is your blueprint for the report card. The +kubebuilder:subresource:status marker is a crucial part of this. It tells the Kubernetes API that this custom resource has a status field that should be managed separately from the spec.
Go
```
// In api/v1alpha1/trinocluster_types.go

// TrinoClusterStatus defines the observed state of TrinoCluster.
type TrinoClusterStatus struct {
    // Phase is a simple, high-level status field.
    Phase string `json:"phase,omitempty"`
    
    // ReadyReplicas is the number of ready pods in the deployment.
    ReadyReplicas int32 `json:"readyReplicas,omitempty"`

    // The URL of the coordinator service.
    CoordinatorURL string `json:"coordinatorURL,omitempty"`
}

// +kubebuilder:object:root=true
// +kubebuilder:subresource:status
type TrinoCluster struct {
    metav1.TypeMeta   `json:",inline"`
    metav1.ObjectMeta `json:"metadata,omitempty"`

    Spec   TrinoClusterSpec   `json:"spec,omitempty"`
    Status TrinoClusterStatus `json:"status,omitempty"`
}
```
Step 2: Collect the Observed State
Next, you need to add logic to your Reconcile function to collect the observed state of the cluster. A great way to do this is to check the status of the resources you've already created.
Go
```
// In the Reconcile function, after reconciling your deployments

// Get the status of the coordinator deployment.
coordinatorDeployment := &appsv1.Deployment{}
if err := r.Client.Get(ctx, types.NamespacedName{...}, coordinatorDeployment); err != nil {
    // handle error
}

// Get the status of the worker deployment.
workerDeployment := &appsv1.Deployment{}
if err := r.Client.Get(ctx, types.NamespacedName{...}, workerDeployment); err != nil {
    // handle error
}
```

// Now, update the status fields in your TrinoCluster CR.
```
trinoCluster.Status.ReadyReplicas = coordinatorDeployment.Status.ReadyReplicas + workerDeployment.Status.ReadyReplicas
trinoCluster.Status.Phase = "Running"
trinoCluster.Status.CoordinatorURL = fmt.Sprintf("http://%s-coordinator-service:8080", trinoCluster.Name)
Step 3: Update the Custom Resource
The final, crucial step is to write the updated status back to the Kubernetes API. The controller-runtime framework provides a dedicated, clean way to do this. The r.Client.Status().Update() method is designed to update only the status field of a resource. This is a best practice that prevents race conditions and ensures that your operator's reconciliation loop is predictable.
Go
// In the Reconcile function, after collecting the status
if err := r.Client.Status().Update(ctx, trinoCluster); err != nil {
    log.Error(err, "Failed to update TrinoCluster status")
    return ctrl.Result{}, err
}
```
