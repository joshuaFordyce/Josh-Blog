---
title: 'Gateway API for Trino Part 5 '
description: 'In this article, we analyzed the results of our Gateway API vs Ingress experiment'
pubDate: 'Feb 24, 2026'
category: 'Data Engineering and Data Infrastructure'
heroImage: '../../assets/images/2 - FA6Yb.jpg'
tags: ['ML']
---

In this article we’ll chat about the results of our experiment. We hypothesized that the Kubernetes Gateway API would maintain session persistence during route mutations due to its atomic update model while the Nginx Ingress Reloads would sever TCp connections, leading to query failures

Experiment Design

We developed a python script to simulate  a flapping infra while executing complex SQL on a Trino Cluster

Asynchronous Trino queries via threading to simulate active user sessions
We used subprocess to flap a legacy dummyingress.yaml and the NetworkingV1Api to delete it mid-query
We used CustomObjectsAPI to perform JSON patches on httproutes, which allowed us to add and remove backendRefs dynamically to test delta-updates


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


Results of our experiment

Let's talk about what we saw during our experiment. 

Data


K8s networking Object








Ingress (Nginx)
P99 latency spikes during reload
Grew linearly and reparsing took longer as we added routes
Worker’s weren’t updated atomically


Gateway API
Zero-downtime 
Updates were incremental and only the delta was sent so the config size of the httproute didn’t grow linearly
Atomic updates across the fleet





Interpreting the Results

Our data shows that the scalability of Ingress or lack thereof is liability for high performant workloads running on kubernetes. A simple routing change for one service shouldn’t cause a P99 spike for another service. This disrupts the user experience severely for the end users who depend on that service. 

Our data also shows that Gateway API allows us to decouple the growth of our infra from the latency of our traffic by doing incremental updates. This helps in situations where we’re doing real time analytics that rely on high performant writes. A 200ms reload spike due to us updating a route would be unacceptable in this situation.

Conclusion

Our data shows that the lack of scalability of Ingress is a liability for high-performance workloads. By decoupling the growth of the infra from the latency of our traffic we don’t cause a P99 spike for your query engine. 
