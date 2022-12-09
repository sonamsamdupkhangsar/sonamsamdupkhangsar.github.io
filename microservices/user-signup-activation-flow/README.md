# User Signup Workflow
The following shows the user signup workflow using the user-rest-service, authentication-rest-service and account-rest-service.


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
