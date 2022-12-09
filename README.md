# sonamsamdupkhangsar.github.io

### integration of microservices
I have been working on some microservices developed using Java with Spring Reactive.
These services are in my github repositories.

I currently have the services deployed on a Kubernetes cluster using a Nginx Ingress controller.  

### Kubernetes cluster
The following diagram shows the request flow on a Kubernetes cluster.

```mermaid
flowchart TD
    A[user request] -.-> B(Load balancer)
    B -.-> C(DNS Server)
    C -.-> D[/email-rest-service.sonam.cloud/]

    subgraph k8[Kubernetes Cluster]
    subgraph ingress[Ingress]
    F(Nginx Controller)
    end

    subgraph app[email-rest-service]
    G(Kubernetes Service)
    G -- uses authId header for user context --> H(email-rest-service pod)
    H -- validate jwt token using jwt-validator --> H
    end    
    
    end
    F -. 3 calls-service .-> G
    B -.-> k8
    
```

Articles

1.  [How to use Maven dependency from Github repository and in your project](/pulling-down-github-maven-library/README.md)
2. [How to build custom authentication with Nginx Ingress](/custom-nginx-authentication-with-auth-url-annotation/README.md)