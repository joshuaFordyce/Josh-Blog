---
title: 'Lets dig deep into Autoscaling'
description: 'In this article, how native Autoscaling works in Kubernetes'
pubDate: 'August 28, 2025'
category: 'Data Engineering and Data Infrastructure'
heroImage: '../../assets/images/2 - FA6Yb.jpg'
tags: ['ML']
---








Background information about HPA. 
The HPA is a kubernetes API resource that will automatically scale the amount of pods in a Deployment, ReplicaSet or Statefulset based on resource utilization or custom metrics. The idea is that you should be able to scale you application to meet demand by maintaining an average utilization target across the Pods

HPA Internals: Control Loop 

The HPA is a dedicated controller that is constantly observing and ready to react to changes in the actual cluster and it itself is controlled by the kube-controller-manager. The HPA Controller exists inside the kube-controller-manager and that is the brain of the HPA. The Metrics Server is a cluster add-on that collects  resource metrics from Kubeletes and uses the Kubernetes API to expose them. The HPA will then query the Metrics Server to obtain these values. The kube-apiserver is the central API endpoint where HPA objects are actually defined. The Kube-apiserver is also where the HPA controller reads its config and where it writes the scaling decisions. The kubelet is the agent on each node that will report Pod resource usage to the Metrics Server.

The Reconciliation Loop is a fundamental design pattern that is implemented by controllers. It continually Observers, Compares, and then Reconciles. The Observer state is where it monitors the current state of specific resources in the Kubernetes cluster. It then compares this actual state against the desired state that we have defined using currentMetricValue vs desiredMetricValue. The formula is desiredReplicas= ceil(currentReplicas * (currentMetricValue / desiredMetricValue)). If 5 Pods are 60% CPU, and our target is 30% CPU then we’d do ceil(5 * ( 60% /3%)) = ceil(5 * 2) = 10 desired replicas. Then the Reconciliation loop will move into the action stage. If the desiredReplicas is greater/less than currentReplicas then the HPA controller will update the replicas field of the target Deployment/StatefulSet resource. The Deployment/StatefulSet controller will then see the updated replicas count and creates or deletes Pods accordingly

The HPA uses a cooldown period that defaults to 5 minutes for scaling down and 3 minutes for scaling up to prevent rapid scaling. It can also consider recent CPU peaks when scaling down to avoid immediate scale up.

How HPA and Starburst Worker Nodes work?

In a Starburst Triono Cluster you typically have a coordinator node that parses queries, plans execution and manages workers. Then the Worker nodes are the actual engines that perform the distributed ata processing. They are typically managed by a Kubernetes Deployment/StatefulSets. When you configure an HPA for the Starburst worker nodes, the HPA object will then target the Kubernetes Deployment that manages the Worker Pods. The HPA will scale the number of worker Pods up and down to directly adjust the query processing capacity of your cluster. 

HPA’s benefits to a Kubernetes based Starburst Deployment are numerous. HPA scales down when not needed and automatically scales up to meet increased query demand, preventing slowdowns and potential failures due to insufficient processing power. There are some very interesting challenges. Choosing metrics that make sense is important. For example, High CPU might indicate efficient processing but Trino queries are typically memory-bound or I/O-bound. A long query queue might show a bottleneck that's not actually reflected by CPU if workers are waiting on I/O. Custom metrics like query queue depth, average query latency, active query count, and time spent in blocked state. New Starburst Worker pods might need time to initialize, connect to the coordinator and then warm up caches before coming online. THis can then add a slight delay to the scale-up  response. Because of the inherent lag between demand increasing and new pods becoming ready, sudden sharp spikes in query load might briefly overwhelm the cluster before the HPA is able complete the scale out process. The HPA only scales workers so the Starburst coordinator has to be able to handle the increased number of worker connections and query planning complexity when scaling the workers.

Troubleshooting HPA

Drop CPU Threshold
  Purpose: This is a way to verify if the HPA is fundamentally working. By setting a very low target you’re able to run a very small load that should trigger scaling
  What do do if it does’t scale
    Check the Metrics server to ensure its not down or misconfigured
      - Kubectl top Pod
        - What it shows
          - It shows real-time CPU and memory usage of the Pods

        - Diagnosis
            - This helps us verify that the Metrics Server is reporting.  If kubectl top shows low cpu but HPA isn’t scaling, there might then be a discrepancy in how HPA reads  metrics
    Check HPA events
      - Kubectl get hpa
        - What it shows:
            - A summary including Name, Targets(current metic vs target), MINPODS, MAXPODS, REPLICAS(current replicas), and AGE
        - Diagnosis
            - Helps confirm the HPA’s current state and target
      - Kubectl describe hpa
        - What it shows:
            - This provides detailed information about the HPA, including the config, status, a nd a list of Events at the bottom
        - Diagnosis
            - The Events section is the most important part. It will tel you why the HPA is or isn’t scaling ( "SuccessfulRescale," "NoPodsAvailableForMetrics," "TooManyReplicas," "DesiredReplicasTooHighForMinAndMax," "BackoffScaleDown"). This is key to understanding why it's not triggering.

If the HPA is increasing, Deployment replicas and pods are stuck in pending , that means there aren’t any available nodes. The Cluster Autoscaler is involved here. For this investigation, you’d check the configs and logs to see if its correctly linked to the Auto Scaling Group and its attempting to scale out nodes


