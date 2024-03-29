# Building Spring WebFlux Application
I enjoy using [Spring WebFlux](https://docs.spring.io/spring-framework/reference/web/webflux.html) for building web applications.  I like learning new ways of programming and sometimes that can come with frustrations too.  The good thing is that there is [StackOverFlow](https://stackoverflow.com/) to ask  or research similar questions.  I would say that Spring WebFlux improves a lot on the existing Servlet based way of building webapps because it provides asynchronous and message based programming to provide higher throughput of request/response.

## Demo `Person` Project
For today I am going to build a `Person` rest service that interacts with a database to fetch person data.  This session will demonstrate how to build a web application using Spring WebFlux.

I also created a Youtube video for this demo which you can watch as well.
[![Youtube demo](https://img.youtube.com/vi/t1hPc8pDbAU/0.jpg)](https://youtu.be/t1hPc8pDbAU)


This project is built using Gradle and can be found on [my git repo](https://github.com/sonamsamdupkhangsar/person).

### Project inventory
Lets inventorize the things we need to build for this webapp.

Things we are going to build:

1. A Rest endpoint using Route construct with 4 endpoints:
    * POST `/persons` to create a new Person entity.
    * PUT `/persons` to update a Person entity.
    * GET `/persons/{id}` to get a Person entity by id.
    * GET `/persons` to fetch a page of persons.

2. A Handler that calls the business service and handles the error from it.
3. A business service to retrieve and store person information.
4. Repository interface that interact with the database.
5. Database Schema


### Project Dependencies

For project dependencies, I use the following: 
1. Spring Webflux .
2. H2 in-memory reactive database driver for integration testing.
3. Postgresql reactive database driver for local running.
4. Junit-Jupiter for test framework.



### Rest Endpoint using Route
For Rest endpoint you can use the `RouterFunction` to build routes or api endpoints.  The following is the method to build a route endpoint taken from [Router](https://github.com/sonamsamdupkhangsar/person/blob/96e55aeba571c8cbe0b9391912f39ca544636ee1/app/src/main/java/org/mycompany/Router.java#L21) configuration:

```java
@Bean
    public RouterFunction<ServerResponse> route(Handler handler) {
        return RouterFunctions.route(GET("/persons/{id}"), handler::getPerson)
        ...
    }
```

In this example we are creating a route to a `/persons/{id}` endpoint that is a Http GET.  The handler method getPerson is called for this endpoint.  

### Building the Handler
The Handler will be used to handle the endpoint and call the business service method.  It can also add a layer to do any error handling as well.

The following example shows the [handler](https://github.com/sonamsamdupkhangsar/person/blob/96e55aeba571c8cbe0b9391912f39ca544636ee1/app/src/main/java/org/mycompany/Handler.java#L27) for the `getPerson()` method:

```java
  public Mono<ServerResponse> getPerson(ServerRequest serverRequest) {
        LOG.info("get person by id");

        return personFinder.getById(UUID.fromString(serverRequest.pathVariable("id")))
                .flatMap(person -> ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).bodyValue(person))
                .onErrorResume(throwable -> {
                    LOG.error("get person by id failed, error: "+ throwable.getMessage());
                    return ServerResponse.badRequest().contentType(MediaType.APPLICATION_JSON)
                            .bodyValue(Map.of("error", throwable.getMessage()));
                });

    }
```
In this handler method example, I use the `personFinder` business service interface to call the `getById` method and flatmap the person obejct to return back the object to caller.  In the even of an exception, I send a BadRequest exception to client.

### Business Service 
The Java Interface `PersonFinder` is used by the handler above to call `getById` method.
The following implementation of the `PersonFinder` shows the `getById` [implementation](https://github.com/sonamsamdupkhangsar/person/blob/795aa94539256a43a3ed9df09903393ac0dc1dd2/app/src/main/java/org/mycompany/SimplePersonFinder.java#L26).

```java
 @Override
    public Mono<Person> getById(UUID id) {
        LOG.info("get a person by id {}", id);

        return personRepository.findById(id).switchIfEmpty(Mono.error(new RuntimeException("No person with id: "+ id)));
    }
```
When this method is called the `personRepository` is used to query the database reactively.  If there are no rows found then it will throw back a RuntimeException using the `switchIfEmpty`.  


### Repository Interface
The repository interface is used to specify the interface for working with the database table. 
The following is the [`PersonRepository`](https://github.com/sonamsamdupkhangsar/person/blob/96e55aeba571c8cbe0b9391912f39ca544636ee1/app/src/main/java/org/mycompany/db/repo/PersonRepository.java#L10C1-L14C2)

```java
public interface PersonRepository extends ReactiveCrudRepository<Person, UUID> {
    //get a page of person with a custom query
    @Query("select * from Person")
    Flux<Person> findAll(Pageable pageable);
}
```

In this example, the PersonRepository interface is created which is a reactive crud capable repository for working with [`Person`](https://github.com/sonamsamdupkhangsar/person/blob/96e55aeba571c8cbe0b9391912f39ca544636ee1/app/src/main/java/org/mycompany/db/repo/Person.java#L9) entity and its primary key is of type UUID.
You can also specify custom sql query using the `@Query` annotation in your repository interface.

### Table creation
I have included a [`schema.sql`](https://github.com/sonamsamdupkhangsar/person/blob/96e55aeba571c8cbe0b9391912f39ca544636ee1/app/src/main/resources/schema.sql#L1) file for creating `Person` table.  This schema file will be executed by the [MySpringApplication](https://github.com/sonamsamdupkhangsar/person/blob/96e55aeba571c8cbe0b9391912f39ca544636ee1/app/src/main/java/org/mycompany/MySpringApplication.java#L19) class.

### Integration Testing endpoints
I have included a [test case](https://github.com/sonamsamdupkhangsar/person/blob/795aa94539256a43a3ed9df09903393ac0dc1dd2/app/src/test/java/org/mycompany/PersonRestIntegTest.java#L32) that calls each of the endpoints in the project.

### Running locally
You will need to create a [`application.yaml`](https://github.com/sonamsamdupkhangsar/person/blob/main/app/src/main/resources/application.yaml) file that points to your local Postgresql database.
```yaml
  r2dbc:
    url: r2dbc:postgresql://localhost:5432/person
    username: test
    password: test
    properties:
      sslMode: disable
```
Once you have the database created, use your username and password in the config above.  The `sslMode` property needs to be set in your config.

To run this project locally use ` ./gradlew bootRun` command.  You should see similar output as below (some lines from output has been redacted):
```java
> Task :app:bootRun

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v3.1.0)

...
2023-11-28T09:47:50.639-07:00  INFO 19763 --- [           main] o.s.b.web.embedded.netty.NettyWebServer  : Netty started on port 8080
2023-11-28T09:47:50.644-07:00  INFO 19763 --- [           main] org.mycompany.MySpringApplication        : Started MySpringApplication in 2.24 seconds (process running for 2.599)
<==========---> 80% EXECUTING [30s]
> :app:bootRun
```

### Testing locally
To call each of the endpoints you can follow this:

1. Use `POST` to create a person by curl: 
```
curl -X POST http://localhost:8080/persons -H "Content-Type: application/json" -d '{"firstName": "Tashi", "lastName": "Tsering"}'
``` 

2. Use `PUT` to update the person by curl (use the id retrieved from the curl response from above): 
```
curl -X PUT http://localhost:8080/persons -H "Content-Type: application/json" -d '{"id": "<REPLACE_UUID_ID_HERE>", "firstName": "Tenzing", "lastName": "Lhamo"}'
```
   

3. Use `GET` to retrieve by id:
```
curl http://localhost:8080/persons/<REPLACE_UUID_ID_HERE>
```
4. Use `Get` persons by page:
```
curl  "http://localhost:8080/persons?page=0&size=100"
```


### Conclusion
In this Readme.md file I showed how to create a Router, Handler, Business service and a Repository interface for building a Spring WebFlux application.   It uses H2 in-memory database for testing purposes but you can also use an external Postgresql database.  

This is a simple webapplication to give a easy introduction to Spring Webflux.  

