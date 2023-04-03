# sonamsamdupkhangsar.github.io
Some links

1. My [Helm chart](https://github.com/sonamsamdupkhangsar/sonam-helm-chart).
2. Some articles of mine on [medium](https://medium.com/@sonamhava) 
3. My messaging app [kecha](https://kecha.sonam.cloud)
4. [How to use Maven dependency from Github repository and in your project](/pulling-down-github-maven-library/README.md)
5. [How to build custom authentication with Nginx Ingress](/custom-nginx-authentication-with-auth-url-annotation/README.md)
6. [Catalogue Rest API with SwaggerUI](./rest-api-catalog-swaggerui/README.md)
7. [OpenApi Rest design](./restapi-spec-with-openapi/README.md)
8. [Consumer driven contract testing](./rest-api-contract-driven-testing/README.md)
9. [Building custom Yaml properties in Spring based application](/java-spring-stuff/building-custom-yaml-properties/README.md)
10. [Retrieve the custom error message from a webservice call on Spring Reactivce stack](/java-spring-stuff/get-error-message-in-webclient-error/README.md)
11. [Create custom page type in a Rest call on Spring Reactive stack](/java-spring-stuff/map-page-response-from-webclient/README.md) 

Some personal stuff I am working on:

1. [User signup flow](/microservices/user-signup-activation-flow/README.md) 

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