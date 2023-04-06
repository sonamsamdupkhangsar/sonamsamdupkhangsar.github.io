# Service Registry and Discovery for Microservices
This document shows how service registry and service discovery works.  It will use a Spring based Java application with Netflix's Eureka registry service.   It will show how clients and microservices can use discovery service for communicating with each other, without needing to know the actual ip or dns addresses.

The following is a diagram of service registeration and service discovery:



```mermaid
flowchart TD   
    A1[Microservice A instance 1]
    A2[Microservice A instance 2]
    B[Microservice B]
    C[Microservice C]
    Eureka[Service Registry]
    D[Client service] 

    A1 --> |register| Eureka
    A2 --> |register| Eureka
    B --> |register| Eureka
    C --> |register| Eureka
    Eureka <--|hello|> D
``` 
    Eureka ---|lookup Microservice A| D

## Service Registry
Service registry is a framework used for registering services.  For example, a microservice can register themselves to a Eureka server.  Once registered the services can show up on the service registry using their configured name such as `hello-world-service` or by any other name. 

## Service Discovery
Service discovery is the process by which the clients will use the registry to connect to a instance of a microservice using a name.


## How does registry and discovery work in Spring based application using Eureka server?
In a Spring based application that uses Eureka registry there will need to be a Eureka server that will be running.
This Eureka server will act as the registry server where all the microservices will registry themselves.


