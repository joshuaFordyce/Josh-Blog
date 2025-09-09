



The Pod That Knew Too Much
How to Pull Kubernetes Metrics from Inside a Container
In a multi-service container environment, a critical challenge is gaining visibility into resource usage. For an application running inside a Kubernetes pod, the most common solution, kubectl top pod, is useless. The kubectl binary is not included in standard container images, so you cannot simply run the command. This leads to a fundamental question: if the client tool is missing, how does one get a pod's CPU and memory usage from within the pod itself?
The solution we discovered lies in directly communicating with the Kubernetes API. The Kubernetes ecosystem is designed for this.
The Role of the Metrics Server
To understand the solution, you must first understand the Kubernetes Metrics Server. It is a lightweight, cluster-wide aggregator of real-time resource usage data. It collects CPU and memory metrics from all nodes and pods and exposes this data via the metrics.k8s.io API. This is the API endpoint that tools like kubectl top query to get their data.
The In-Cluster API Call
When a pod is deployed to a Kubernetes cluster, the system automatically injects a service account token and a certificate authority (CA) into the container's file system. This is done to give the pod's application a secure, authenticated way to communicate with the Kubernetes API server.
We can leverage this mechanism to make a direct API call from our container. The call is a standard curl request to the API server's endpoint, using the automatically mounted credentials for authentication:
APISERVER=https://kubernetes.default.svc: A standard DNS entry that resolves to the API server's internal cluster IP.
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token): The service account token, which acts as a username and password for API requests.
CACERT=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt: The certificate authority file used to verify the API server's identity.
By making this call, we bypassed the need for the kubectl binary entirely. We went directly to the source of the data, which is a more robust and idiomatic way for a containerized application to operate in a Kubernetes environment.
The key lesson here is that in Kubernetes, the API is the foundation, and client-side tools like kubectl are merely a convenience layer on top of it. For true in-container observability, it's often best to talk directly to the API.
