# How to retrieve the custom error message from a webservice call in a error handler

This will show how to access the webservice message when a error occurs during a webservice call.

When you make a webservice all in your code you usually process the response functionally in a map and have a `onErrorResume` handler to handle the call failure.  The following is an example of a http call in a Spring Reactive stack with error handler:

```
   WebClient.ResponseSpec responseSpec = webClient.post().uri(jwtAccessTokenEndpoint)
                .headers(httpHeaders -> httpHeaders.add(HttpHeaders.AUTHORIZATION, hmac))
                .bodyValue(jsonString)
                .accept(MediaType.APPLICATION_JSON)
                .retrieve();
        return responseSpec.bodyToMono(Map.class).map(map -> {
            // do whatever is needed if a call was success
        }).onErrorResume(throwable -> {
            //handle error to get the custom http error message            
            String errorMessage = throwable.getMessage();

            if (throwable instanceof WebClientResponseException) {
                WebClientResponseException webClientResponseException = (WebClientResponseException) throwable;
                LOG.error("error body contains: {}", webClientResponseException.getResponseBodyAsString());
                errorMessage = webClientResponseException.getResponseBodyAsString();
            }
            return Mono.error(new SecurityException(errorMessage));
        });
```    

In the event of a error when a webservice call is made you would have to cast the `Throwable` object into a  `WebClientResponseException` class to get the actual webservice message.  In the above example, you can see the throwable object is casted to a WebClientResponseException to get the actual mesage.  For an example of this usage you can follow this [example](https://github.com/sonamsamdupkhangsar/jwt-validator/blob/0326c2fd1e22645c2c051fca6f504aaad0072eba/src/main/java/me/sonam/security/headerfilter/ReactiveRequestContextHolder.java#L135) from the []`jwt-validator`](https://github.com/sonamsamdupkhangsar/jwt-validator) project.