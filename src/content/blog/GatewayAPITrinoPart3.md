---
title: 'Gateway API for Trino Part 3 '
description: 'In this article, we explored benchmarking Ingress vs gateway api for Trino'
pubDate: 'Jan 19, 2026'
category: 'Data Engineering and Data Infrastructure'
heroImage: '../../assets/images/2 - FA6Yb.jpg'
tags: ['ML']
---

Gateway API for Trino Part 3: The Performance Verdict

Thesis

There should be several architectural benefits of the Gateway API over the Ingress. We’ll use the complex requirements of our Trino workload as a way to benchmark these two kubernetes objects. Trino provides an opportunity to analyze how Ingress-NGINX and our Gateway API manages long-lived high bandwidth TCP streams or 10 minute federated joins.

Methodology

We’ll focus on comparing the legacy Ingress-NGINX controller against an Istio Controller based Gateway API implementation

Legacy Standard Ingress Example

````
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: trino-ingress
  annotations:
    # Vendor-specific annotations are non-portable
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "15"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "600"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "600"
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
            name: trino-coordinator
            port:
              number: 8080


Modern Gateway API

# 1. Infrastructure Template
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: istio-trino-class
spec:
  controllerName: istio.io/gateway-controller
---
# 2. The Entry Point (Load Balancer)
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: trino-gateway
  namespace: trino-namespace
spec:
  gatewayClassName: istio-trino-class
  listeners:
  - name: http
    protocol: HTTP
    port: 80
    allowedRoutes:
      namespaces:
        from: Same
---
# 3. The Query Logic & Timeouts
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: trino-route
  namespace: trino-namespace
spec:
  parentRefs:
  - name: trino-gateway
  hostnames:
  - "trino.example.com"
  rules:
  - matches:
    - path: { type: PathPrefix, value: / }
    backendRefs:
    - name: trino-coordinator
      port: 8080
    # Native, standardized fields for data workloads
    timeouts:
      request: 600s
      backendRequest: 580s

````

Background info on Latency


Experimentation

Config Churn

This script will help us test the static reloads of the Ingress controller versus the Dynamic Streaming that the Gateway API uses. We’re expecting the Ingress to perform worse on this test since reloads is where throughput tends to dip and connections often drop. The P99 Latency for the Ingress should be at least 3x times higher than average latency.

````
import time
import threading
import requests
import statistics
from kubernetes import client, config

config.load_kube_config()
custom_api = client.CustomObjectsApi()
networking_api = client.NetworkingV1Api()

# Experiment Settings
NAMESPACE = "default"
TEST_DURATION = 60 # seconds
CHURN_INTERVAL = 2 # seconds

def ingress_churn():
    """Continuously flips an annotation on a dummy Ingress to force reloads."""
    while True:
        meta = {"name": "benchmark-churn-ingress", "annotations": {"test": str(time.time())}}
        body = client.V1Ingress(
            metadata=client.V1ObjectMeta(name="churn-ing", annotations=meta["annotations"]),
            spec=client.V1IngressSpec(rules=[client.V1IngressRule(host="churn.test", http=client.V1HTTPIngressRuleValue(paths=[client.V1HTTPIngressPath(path="/", path_type="Prefix", backend=client.V1IngressBackend(service=client.V1IngressServiceBackend(name="trino-svc", port=client.V1ServiceBackendPort(number=8080)))]))]))
        )
        # Apply change
        networking_api.patch_namespaced_ingress("churn-ing", NAMESPACE, body)
        time.sleep(CHURN_INTERVAL)

def gateway_churn():
    """Continuously flips a filter on an HTTPRoute to force xDS updates."""
    while True:
        body = {
            "apiVersion": "gateway.networking.k8s.io/v1",
            "kind": "HTTPRoute",
            "metadata": {"name": "churn-route"},
            "spec": {
                "parentRefs": [{"name": "trino-gateway"}],
                "rules": [{"filters": [{"type": "RequestHeaderModifier", "requestHeaderModifier": {"add": [{"name": "churn", "value": str(time.time())}]}}],
                           "backendRefs": [{"name": "trino-svc", "port": 8080}]}]
            }
        }
        custom_api.patch_namespaced_custom_object("gateway.networking.k8s.io", "v1", NAMESPACE, "httproutes", "churn-route", body)
        time.sleep(CHURN_INTERVAL)

def run_latency_test(url, label):
    latencies = []
    errors = 0
    end_time = time.time() + TEST_DURATION
    
    print(f" Starting {label} Throughput Test...")
    while time.time() < end_time:
        start = time.time()
        try:
            resp = requests.get(url, timeout=1.0)
            if resp.status_code == 200:
                latencies.append((time.time() - start) * 1000)
            else:
                errors += 1
        except:
            errors += 1
        time.sleep(0.05)
    
    print(f"\n--- {label} RESULTS ---")
    print(f"Avg Latency: {statistics.mean(latencies):.2f}ms")
    print(f"P99 Latency: {statistics.quantiles(latencies, n=100)[98]:.2f}ms")
    print(f"Connection Errors/Drops: {errors}")

# To run: Uncomment one churn and the corresponding test
# threading.Thread(target=ingress_churn, daemon=True).start()
# run_latency_test("http://trino-legacy.example.com/v1/info", "INGRESS")


````
Large Headers
Trino is notorious for large headers and to solve this Ingress NGINX requires manual tuning of proxy-buffer-size via annotations to accept them. Whereas the Gateway API handles larger defaults more gracefully.

````
#!/bin/bash
# Generate a 16KB Header (typical auth token size)
HEADER_DATA=$(head -c 16384 < /dev/urandom | base64 | tr -d '\n')

benchmark_throughput() {
    local URL=$1
    local NAME=$2
    echo "Benchmarking $NAME with 16KB Headers..."
    # Use 'hey' for high-concurrency throughput testing
    hey -n 2000 -c 50 -H "X-Trino-Token: $HEADER_DATA" "$URL" | grep -E "Requests/sec|99%"
}

# Run both
benchmark_throughput "http://trino-legacy.example.com/v1/info" "Legacy Ingress"
benchmark_throughput "http://trino.example.com/v1/info" "Gateway API"
````



We expect that the Large Header test will return a 413 Entity Too Large or a 502 Bad Gateway because the annotation tax of Ingress requires that we add more YAML annotations to make the test pass whereas the Gateway API should just work immediately.



Query Resiliency

Trino queries can explode in duration and Ingress controllers thar reload typically reset long-lived TCP connections which can kill the query. In this script we run a massive join and while it’s running we update a different route in the cluster. Standard Ingress controllers like NGINX use a Process model in which changing a config causes the “Master” process spawns new “Workers” and tells older ones to shut down. If the old worker is holding the long running Trino query, it may run into a worker-shutdown-timeout and this kills the socket. We want to focus on whether the query actually completed and then how long the connection stayed open.


Script

````
import trino
import time
import threading


from kubernetes import client, config
import requests
import statistics
from trino.dbapi import connect
import kubernetes.client
import subprocess
import time


from kubernetes.client.rest import ApiException


config = config.load_kube_config()
custom_api = client.CustomObjectsApi()
networking = client.NetworkingV1Api(config)




def configureConnection(host):


   conn = connect(
       host=host,
       port="80",
       user="admin",
       catalog="tpch",
       schema="sf1",
       request_timeout=60.0
   )
   cur = conn.cursor()
   return cur


def executeQuery(host,i):
   start_time = time.time()
   try:
       query = """
       SELECT
       nation,
       o_year,
       sum(amount) AS sum_profit
   FROM
       (
           SELECT
               n_name AS nation,
               extract(YEAR FROM o_orderdate) AS o_year,
               l_extendedprice * (1 - l_discount) - ps_supplycost * l_quantity AS amount
           FROM
               part,
               supplier,
               lineitem,
               partsupp,
               orders,
               nation
           WHERE
               s_suppkey = l_suppkey
               AND ps_suppkey = l_suppkey
               AND ps_partkey = l_partkey
               AND p_partkey = l_partkey
               AND o_orderkey = l_orderkey
               AND s_nationkey = n_nationkey
               AND p_name LIKE '%green%'
       ) AS profit
   GROUP BY
       nation,
       o_year
   ORDER BY
       nation,
       o_year DESC;
       """
       cursor = configureConnection(host)
       cursor.execute(query)
       rows = cursor.fetchall()
       end_time = time.time()
       print(f"SUCCESS: In-flight query: {i} survived! (Took  {end_time - start_time:.2f}s)")
       test_results.append(True)
   except Exception as e:
       fail_time = time.time()
       print(f"FAILURE: Query:{i} severed at {fail_time - start_time:.2f}s. Error: {e}")
       test_results.append(False)
  
def delete_ingress():


   try:


       api_response = networking.delete_namespaced_ingress(
           name="minimal-ingress",
           namespace="default",
           body=client.V1DeleteOptions(
               propagation_policy='Foreground',
               grace_period_seconds=5
           )
       )
       print("deleting ingress")
       return api_response
  
   except Exception as e:
       pass


def create_Ingress():


   bash_code = """
       kubectl apply -f dummyingress.yaml
   """
   subprocess.run(bash_code, shell=True)




def add_route():
## finish this portion of this
   #api_instance = kubernetes.client.CustomObjectsApi(config)
   group = "gateway.networking.k8s.io"
   version = "v1"
   plural = "httproutes"
   name = 'trino-api-route'
   namespace= "gateway-system"
   while True:
       try:
           route = custom_api.get_namespaced_custom_object(
               group, version, namespace, plural, name
           )


           backends = route['spec']['rules'][0].get('backendRefs', [])
  
      
           if not any(backend.get('name') == 'trino-worker-service' for backend in backends):


               backends.append({"name": "trino-worker-service", "port": 8081, "kind": "Service"})
      
   #dry_run = ''
   #field_manager = ''
   #field_validation = ''
   #force = True
  
               api_response = custom_api.patch_namespaced_custom_object(group,version, namespace,plural, name,body=route)
          
           #print(api_response)
               return api_response
           else:
               print("Backend still exists meaning the delete didn't work")
       except ApiException as e:
           if  e.status == 409:
          
               print(f"Exception when calling Doing Add operation CustomObjectsApi: {e}")
               time.sleep(10)
           else: raise e


def delete_route():
## finish this portion of this
   #api_instance = kubernetes.client.CustomObjectsApi(config)
   group = "gateway.networking.k8s.io"
   version = "v1"
   plural = "httproutes"
   name = 'trino-api-route'
   namespace= "gateway-system"
   while True:
       try:
           current_route = custom_api.get_namespaced_custom_object(
               group, version, namespace, plural, name
           )
  
           current_list = current_route['spec']['rules'][0].get('backendRefs',[])
           new_list = []
           for backend in current_list:
               if backend.get('name') != 'trino-worker-service':
                   new_list.append(backend)
  
           current_route['spec']['rules'][0]['backendRefs'] = new_list
  
           body = current_route
   #dry_run = ''
   #field_manager = ''
   #field_validation = ''
   #force = True
  
           api_response = custom_api.patch_namespaced_custom_object(group,version, namespace,plural, name,body)
          
           return api_response
       except ApiException as e:
           if e.status == 409:
               #retry_count += 1
               print(f"Exception when doing delete operation: {e}")
               time.sleep(10)
           else:
               print(f"Exception when doing delete operation: {e}")
      


def Ingress_Stability():
   #Point the query at the Ingress endpoint
   global test_results
   test_results = []
   for i in range(10):


       create_Ingress()
       time.sleep(30)


       thread = threading.Thread(target=executeQuery, args=("trino-legacy.example.com", i))
       thread.start()


       time.sleep(30)
      
      
       print(f"Starting Flap iteration: {i+1}")
       delete_ingress()
      
       thread.join()
   success = test_results.count(True)
   print(f"final results -  Success:{success}/{len(test_results)}")


def Gateway_Stability():
   #Point the query at the Gateway API endpoint
   global test_results
   test_results = []
   for i in range(10):


       add_route()
       time.sleep(30)
       thread = threading.Thread(target=executeQuery, args=("trino.example.com", i))
       thread.start()


       time.sleep(30)


       print(f"Starting Flap iteration: {i+1}")
       #add_route()
       delete_route()
      
       thread.join()
   success = test_results.count(True)
   print(f"final results -  Success:{success}/{len(test_results)}")


def main():


   print("--- Starting Gateway Stability Test ---")
   Gateway_Stability()


   print("\n--- Starting Ingress Stability Test ---")
   Ingress_Stability()




if __name__ == "__main__":
   main()
````









Through this three part benchmarking suite we’re looking at the physical behavior of data at scale. We’re trying to answer the following questions.

Does the Gateway API help our Trino Cluster survive the “administrative churn” of a busy Kubernetes environment, or  will every config change kill our long running queries
Does the Gateway API help Trino more easily stream gigabytes of results back to the user?
Does the Gateway API do a better job of aiding trained in serving small metadata queries while the system is managing heavy ETL jobs?

In my next article in the series, we’ll dive into a primer on latency and how it affects systems like Trino and Kubernetes.






