


Configuring Multiple Certificate Authorities for a Kubernetes Based Trino Deployment:

- Routing traffic from different client groups to a single Kubernetes service/cluster where each group requires trust from a distinct Certificate Authority can be a complex challenge. We discuss how to approach this from a networking and kubernetes perspective below.

Dual Trust for a Single Endpoint

- We want to serve different client types with different CA trust requirements that require a public CAa nd an internal enterprise CA for the same target. 

The Infeassibility: DNS and TLS Limitations

A single DNS record resolves to one or more specific IP addresses. This is not  dynamically based on the client’s origin or trust chain. During the TLS handshake the server presents a certificate for a given hostname. It can’t dynamically go back and forth between two different CA-signed certificates to present based on the connecting client’s source IP. This means that you can’t serve different CA-signed certificates from the exact same network endpoint for the same hostname to different client groups.

First Workaround: Client-side Override for Differentiation
/etc/hosts File Modification
Internal client machines can modify their local /etc/hosts file to force a specific hostname to resolve to a different, internal IP address
This internal IP would point to a separate internal Load Balancer/Ingress Configured with the specific CA certificate. The External client would continue to use public DNS, resolving the same hostname to an external IP/endpoint with a different CA
Caveats 
Some drawbacks include significant operational overhead, maintenance challenges, lack of scalability for large client bases, and dependence on client- side management


Standard Kubernetes Alternative: Separate Hostnames
Using different hostnames each pointed to its own external endpoint configured with the desired CA.
Example Multi-Host Ingress YAML
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: trino-ingress
spec:
  tls:
  - hosts:
    - quicksight.trino.example.com
    secretName: quicksight-cert
  - hosts:
    - internal.trino.example.com
    secretName: internal-cert
  rules:
  - host: quicksight.trino.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: quicksight-service
            port:
              number: 443
  - host: internal.trino.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: internal-service
            port:
              number: 443
```
