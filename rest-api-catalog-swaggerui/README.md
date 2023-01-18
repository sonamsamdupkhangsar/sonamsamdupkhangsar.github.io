# Rest api catalog with Swagger UI
[Swagger UI](https://swagger.io/tools/swagger-ui/) is used for displaying your Rest api contract.   It can also be used for testing endpoints that are defined on the contract.   


Here is a picture of a Swagger UI with a Rest api contract:
![Swagger UI with a Rest api](images/swagger.png)

<br/>
The API that is displayed on the SwaggerUI can be generated using vendor provided libraries or with Swagger libraries.  Or you can also feed in a contract file into the SwaggerUI so it can catalog the Rest api.  


## Generating Swagger UI contract 
There are two ways of generating the contract that is diplayed on the Swagger UI.  One of them is using the Swagger or vendor provided libraries.  The user can decorate the source code endpoints with Swagger annotations to mark the rest endpoints.  


For example, take the following example for Spring WebFlux project where the route is setup using functional programming:

```
@Configuration
@OpenAPIDefinition(info = @Info(title = "Account rest service Swagger doc", version = "1.0", description = "Documentation APIs v1.0"))
public class Router {
    private static final Logger LOG = LoggerFactory.getLogger(Router.class);

    @Bean
    @RouterOperations(
            {
                    @RouterOperation(path = "/accounts/active/{userId}"
                    , produces = {
                        MediaType.APPLICATION_JSON_VALUE}, method= RequestMethod.GET,
                         operation = @Operation(operationId="activeUserId", responses = {
                            @ApiResponse(responseCode = "200", description = "successful operation"),
                                 @ApiResponse(responseCode = "400", description = "invalid user id")}
                    ))
            }
    )
    public RouterFunction<ServerResponse> route(AccountHandler handler) {
        LOG.info("building router function");
        return RouterFunctions.route(GET("/accounts/active/{userId}")
        ...

```
Here the Swagger endpoint starts with `@OpenAPIDefinition` that marks the API title and version information.  It then uses the RouterOperation to define the endpoints, the type of MediaType response it produces, and response codes.

If you are programming with the servlet model using Springboot then your annotations may look a little different.  However, the idea is the same where the source code is annotated which generates the contract shown on the Swagger UI.

The other method is to create the contract using OpenAPI specification which results in a `openapi.yaml` file for a yaml format.  I prefer this approach of creating the contract spec using a openapi.yaml because it separates out the specification from the codebase.  It also leads to a cleaner code.  Now, the contract can be shared among peers, between consumer and providers and be agreed upon using Github pull requests.  


## Cataloging API
You can use the manually created openapi.yaml contract files and host them on a SwaggerUI locally in your company.  It can be used to provide visibility into the apis and also serves as a catalogue. 
I have used SwaggerUI on Java based projects and I find it easy to host a SwaggerUI using Spring WebFlux project.  

Using the following Spring configuration I can host a collection of apis or openapi.yaml specs:

```

springdoc:
  swagger-ui:
    urls[0]:
      url: /v3/api-docs/myapi/application-rest-service-openapi.yaml
      name: application-rest-service
    urls[1]:
      url: /v3/api-docs/myapi/authentication-rest-service-openapi.yaml
      name: authentication-rest-service
    urls[2]:
      url: /v3/api-docs/myapi/user-rest-service-openapi.yaml
      name: user-rest-service
    urls[3]:
      url: /v3/api-docs/jwt/jwt-rest-service-openapi.yaml
      name: jwt-rest-service
    urls[4]:
      url: /v3/api-docs/email/email-rest-service-openapi.yaml
      name: email-rest-service
```

These apis are then visible locally on my machine:
![swaggerui-run-locally-example](images/swagger-catalogue-api.png)

You can navigate around the apis by selecting each definition listed on the pull down menu. 
