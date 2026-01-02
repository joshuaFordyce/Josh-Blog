---
title: 'Gateway API for Trino Part 1 '
description: 'In this article, we explored the differences between GatewayAPI and Ingress'
pubDate: 'Jan 6, 2026'
category: 'Data Engineering and Data Infrastructure'
heroImage: '../../assets/images/2 - FA6Yb.jpg'
tags: ['ML']
---
Introduction

The Kubernetes Ingress abstraction was designed with simple HTTP routing in mind. However, a system like Trino makes configuring and maintaining the abstraction incredibly difficult. This is for a couple of reasons that include vendor -specific annotations, Lack of built-in TCP/UDP support, and inflexible Traffic Splitting . The new Kubernetes Gateway API is the logical successor to the ingress because it provides solutions to these issues. 

Key Problems with Ingress and how the Gateway API fixes this

Ingress’s Clunky Solution: Vendor Annotations

The Standard Ingress spec is incredibly limited so when a new advanced feature becomes available, its handled using vendor-specific annotations. For example, custom timeouts are great because you can specify how long a load balancer or proxy will wait for a connection to be established or a service to respond before it closes the connection and returns a 504 gateway timeout. When you configured an Ingress typically you deal with three main tiers. The tiers are Connect Timeout, Read Timeout and Send Timeout. This is difficult to configure because you have to use vendor-specific annotations to set it. Once you switch cloud providers or ingress controllers your config is broken leading to a lack of infrastructure portability.

Example
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: trino-ingress
  annotations:
    # NGINX specific - non-portable and hard to validate
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "15"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "600" # 10 mins for big queries
    nginx.ingress.kubernetes.io/proxy-send-timeout: "600"
spec:
  rules:
  - host: trino.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: trino-coordinator
            port:
              number: 8080
```



Gateway Api’s Smooth fix: HttpRoute/BackendTLS Policy

The Gateway API s moves these settings into a structured filter or spec area of the HttpRoute and this allows for Validation so the cluster can tell signal to the engineer if the timeout format. 

Example
```
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: trino-route
spec:
  parentRefs:
  - name: shared-gateway # The "Infrastructure" managed by the DevOps team
  rules:
  - matches:
    - path: { type: PathPrefix, value: / }
    backendRefs:
    - name: trino-coordinator
      port: 8080
    # Timeouts are becoming standardized fields
    timeouts:
      request: 600s
      backendRequest: 580s

```


Ingress’s HTTP Only Design

The original Kubernetes Ingress resource was built specifically for Layer 7 traffic. This assumes every request has a URL, hostname and HTTP headers. The issue with this is that many services don;t speak HTTP. For our purposes, Trino typically is deployed in environments where internal data transfer, health checks and secondary protocols require Layer 4 access. To bridge this gap, you have to edit a global ConfigMap and then restart the controller or you can use a dedicated Service of type: LoadBalancer for every single port. This makes it difficult ot manage non-HTTP traffic through a clean declarative API and causes you to end up with a separate set of rules for data traffic that the standard ingress rules can’t see or touch

Example
	
```	
# This is a global setting that affects EVERYONE
apiVersion: v1
kind: ConfigMap
metadata:
  name: tcp-services
  namespace: ingress-nginx
data:
  "8888": "default/trino-internal-service:8888"
```


Gateway API’s Solution: Protocol-specific Routes
	
The Gateway API was designed to be protocol-agnostic which means that instead of forcing everything into an “ingress” object, it provides a set of “Route” resources that are tailored to specific layers of the OSI Model. The Gateway acts as the entry point and then you attach the appropriate route type to  it
	
Routes
HTTPRoute: For the Standard HTTP Traffic
TCPRoute: For high fidelity TCP connections
UDPRoute: For high speed UDP connections
TLSRoute: FOr “pass-through” encrypted traffic where the Gateway doesn’t decrypt the data it instead just lets the traffic 
 	
Example
```
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: TCPRoute
metadata:
  name: trino-internal-data
spec:
  parentRefs:
  - name: shared-gateway
    sectionName: tcp-8888 # Matches a listener defined on the Gateway
  rules:
  - backendRefs:
    - name: trino-worker-service
      port: 8888
```



Ingress: All or Nothing Routing

The Standard Kubernetes Ingress has no concept of weight. It just focuses on routing traffic to the single backend thats defined. The problem with this is that if you want to route a certain percentage of your Trino traffic to a Canary cluster running a new version, Ingress can’t do that. To do this you have to rely on controller-specific annotations and these hacks are usually binary which means its not simple to split traffic across three different versions without incredibly complex, nested ingress rules.

Stable Ingress
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: trino-stable
  namespace: trino-namespace
spec:
  ingressClassName: nginx
  rules:
  - host: trino.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: trino-v1-stable
            port:
              number: 8080

   ```

Canary Ingress that hijacks a portion of the traffic
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: trino-canary
  namespace: trino-namespace
  annotations:
    # This is the 'magic' that makes it a canary
    nginx.ingress.kubernetes.io/canary: "true"
    # This tells NGINX to send exactly 10% of traffic here
    nginx.ingress.kubernetes.io/canary-weight: "10"
spec:
  ingressClassName: nginx
  rules:
  - host: trino.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: trino-v2-canary
            port:
              number: 8080

```

The Gateway API solves this by introducing Weights as a first-class art of the HTTPRoute spec. ITs no longer a “hack” but the intended design. In this new design, a single rule can have multiple backendRefs with each with its own weight. The controller handles the math and the load balancing automatically.
```
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: trino-canary-route
spec:
  parentRefs:
  - name: shared-gateway
  rules:
  - backendRefs:
    - name: trino-v1-stable
      port: 8080
      weight: 90  # 90% of traffic
    - name: trino-v2-canary
      port: 8080
      weight: 10  # 10% of traffic
```


Migrating from Ingress to the Gateway API is a fundamental shift that allows Platform engineers to treat Trino workloads as a true enterprise-grade data engine. The Gateway API addresses the Trino challenges of long running queries, high-concurrency worker-to-coordinator communication, and zero-downtime requirements. The Gateway API addresses these Trino-specific challenges head-on. By moving away from vendor-specific annotations to standardized HTTPRoute timeouts, we ensure that massive, federated queries aren’t killed by a default timeout in a generic load balancer. The native weighted traffic splitting allows platform teams to roll out new Trino versions using a canary approach which means you can verify the stability of your SQL workers with 5% of the production load before committing the entire cluster. The addition of TCPRoute allows platform engineers to manage inter cluster communication and metadata traffic alongside standard SQL over HTTP traffic under a single API for infrastructure cleanliness.




