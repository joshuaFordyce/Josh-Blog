Starburst Cluster Deployment Upgrade Strategies

Starburst upgrading can be a complex tasks for deploying new versions or significant config changes especially in a production environment. Starburst recommends that Trino Infrastructure is upgraded using a blue-green deployment

Why Blue-Green?

Starburst servers within a single cluster are not able to be configured with different versions simultaneously. This means that all of the components must run the same version to maintain stability and functionality. Therefore, a Rolling updates strategy can be problematic due the fact that it introduces temporary version mismatches.

Process Overview of Load Balanced Clusters:

- Designate the current active production cluster as “Blue”. The Load Balancer directs user traffic to this “Blue” cluster
- Update the inactive “Green” cluster with the new Trino Version and desired configuration changes
- Install the updated configuration and resources on the “Green” cluster, then bring it online
- Thoroughly test the “Green” cluster using its direct IP address or a separate, alternative FQDN configured on the LB
- Once testing is successful, reconfigure the Load Balancer to immediately switch all user traffic from the “Blue” cluster to the “Green” cluster
- Then, after we confirm that the “Green” cluster is stable and handling prod traffic, shut down the “Blue” cluster

Why does Starburst not recommend Rolling Updates:

During a rolling update, individual worker servers will run different versions of Starbursst. This version mismatch can cause workers to fail to communicate properly with the coordinator server
Rolling updates introduce config changes gradually across the cluster and this can lead to inconsistency in the cluster
This can cause production queries to fail unexpectedly which will impact users

Summary:

The blue-green deployment model is the preferred approach for upgrades and major config changes, providing a safer and more stable transition by entirely swapping out one consistent cluster for another.
