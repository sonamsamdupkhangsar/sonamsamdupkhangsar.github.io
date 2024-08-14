# How to retrieve the custom error message from a webservice call in a error handler

This will show how to access the webservice message when a error occurs during a webservice call.

When you make a webservice call in your code you usually process the response functionally in a map and have a `onErrorResume` handler to handle the call failure.  The following is an example of a http call in a Spring Reactive stack with error handler:

```
1. final String endpoint = localHost+"/api/health/throwerror";

2. WebClient.ResponseSpec responseSpec = webClient.get().uri(endpoint).accept(MediaType.APPLICATION_JSON).retrieve();
3. return responseSpec.bodyToMono(Map.class).
        flatMap(s ->
                    ServerResponse.ok().contentType(MediaType.APPLICATION_JSON)
                            .bodyValue(getMap(Pair.of("message", s.toString()))))
    .onErrorResume(throwable -> {
       LOG.error("endpoint: '{}' rest call failed: {}", endpoint, throwable.getMessage());
        String errorMessage = throwable.getMessage();

4.        if (throwable instanceof WebClientResponseException) {
                WebClientResponseException webClientResponseException = (WebClientResponseException) throwable;
                LOG.error("error body contains: {}", webClientResponseException.getResponseBodyAsString());
                errorMessage = webClientResponseException.getResponseBodyAsString();
            }
            return Mono.error(new SecurityException(errorMessage));
        });   
```    

In the event of a error when a webservice call is made you would have to cast the `Throwable` object into a  `WebClientResponseException` class to get the actual webservice message.  In the above example, you can see the throwable object is casted to a WebClientResponseException to get the actual mesage.  For an example of this usage you can follow this [example](https://github.com/sonamsamdupkhangsar/jwt-validator/blob/0326c2fd1e22645c2c051fca6f504aaad0072eba/src/main/java/me/sonam/security/headerfilter/ReactiveRequestContextHolder.java#L135) from the [`jwt-validator`](https://github.com/sonamsamdupkhangsar/jwt-validator) project.

The example I have shown is from a [test case](https://github.com/sonamsamdupkhangsar/jwt-validator/blob/658c420b2459d888c0caf8ac383e83ed20265ea5/src/test/java/me/sonam/security/EndpointHandler.java#L197).  In this example, the proxied service returns a bad request or http 400 error invoking `onErrorResume` block in the caller code.  I print both the `thorwable.getMessage()` vs the `webClientResponseException.getResponseBodyAsString()` output to show the output.

## Test output
When I run the test case, the proxied error http message is captured here:

```
EndpointHandler        : throwing error
2023-04-03 10:41:14.484 ERROR 60979 --- EndpointHandler        : endpoint: 'http://localhost:10001/api/health/throwerror' rest call failed: 400 Bad Request from GET http://localhost:10001/api/health/throwerror
2023-04-03 10:41:14.484 ERROR 60979 --- EndpointHandler        : error body contains: throwing error
```


# Multiple occurances of org.json.JSONOBJECT on the class path:

 testImplementation ('org.springframework.boot:spring-boot-starter-test') {
        exclude (group: 'com.vaadin.external.google', module: 'android-json')
    }
