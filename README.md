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
    end    
    subgraph jwt[jwt-rest-service]
    I(Kubernetes Service)
    I --> J(jwt-rest-service pod)
    end    
    end
    F -. 3 calls-service .-> G
    F -. 1 validate jwt .-> I
    B -.-> k8
    I -. 2 set authId-header .-> F
    
```

#### User Signup diagram
```mermaid
flowchart TD
    UserRequest[user request] --> |1. new user signup| UserRestService[user-rest-service]
    UserRestService -->|2. create user record| UserPgsqlDb[(user postgresqldb)]
    UserRestService -->|3. create Authentication `/public/authentications` | AuthenticationRestService[authentication-rest-service]
    AuthenticationRestService -->|4. save authentication| AuthenticationPgsqlDb[(authentication postgresqldb)]
    UserRestService -->|5. create Account inActive `/accounts/authenticationId_value/email_value`| AccountRestService[account-rest-service internal]
    AccountRestService -->|6. save Account and create passwordsecret| AccountPgsqlDb[(account postgresqldb)]
    AccountRestService --> |7. email user with link to activate account with secret| EmailRestService[email-rest-service internal]    
```

#### User Activate Account diagram
```mermaid
flowchart TD
    UserRequest[user request] -->|1. user activates account with link to AccountRestService url| AccountRestService[account-rest-service]
    AccountRestService --> |2. save account in active state| AccountPgsqlDb[(account postgresqldb)]
```

#### User Authentication diagram

```mermaid
flowchart TD
    UserRequest[user request] --> |1. authenticate with username/password `/public/authentications/authenticate`| AuthenticationRestService[authentication-rest-service]    
    AuthenticationRestService --> |2. check account for active state| AccountRestService[account-rest-service]
    AuthenticationRestService --> |3. create jwt| JwtRestService[jwt-rest-service internal]    
    JwtRestService -. 4. JWT token .-> UserRequest    
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
