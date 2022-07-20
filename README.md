# sonamsamdupkhangsar.github.io

### integration of microservices
I have been working on some microservices developed using Java with Spring Reactive.
These services are in my github repositories.

I currently have the services deployed on a Kubernetes cluster using a Nginx Ingress controller.  


#### User Signup diagram
```mermaid
flowchart TD
    A[user request] --> |new user signup| B[user-rest-service]
    B[user-rest-service] -->|create user record| E[(user postgresqldb)]
    B[user-rest-service] -->|user signup| C[authentication-rest-service]
    C -->|authentication create| D[(authentication postgresqldb)]       
```

#### User Authentication diagram

```mermaid
flowchart TD
    A[user request] --> |authenticate with username/password| B[authentication-rest-service]
    B --> | create jwt| C[jwt-rest-service]    
``` 

#### User Access protected resource diagram
The dashed line indicates the jwt validation that occurs when a request is 
routed to jwt-rest-service by the Nginx Ingress controller.  The redirection
occurs using the Nginx ingress annotation for "auth-url" annotations in the ingress.yaml
file:
```
annotations:
    nginx.ingress.kubernetes.io/auth-url: "https://$host/oauth2/auth"
```

```mermaid
flowchart TD    
    A[user request] --> |http authorization bearer jwt in header| B[Nignx Ingress]
    B -.-> | nginx ingress auth-url jwt validation| C{jwt-rest-service}
    C -.-> |http 200 status| B
    B -->|secured service| D[user-rest-service] 
    C -.-> | http not 200 status| A
``` 
