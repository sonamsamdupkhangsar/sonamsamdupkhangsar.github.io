# how-to do something in Java Spring related
This will contain small bits of code on how to do something that I keep forgetting because I don't do it often.

## How to parameterize types in bodyToMono
When you have want to parameterize type into the `bodyToMono(Map.class)` call because you want to support the actual type that is in the Map.  You can use `ParameterizeTypeReference` in the `bodyToMono()` call.  The following is an example of using `ParameterizeTypeReference` :

```
@Override
    public Mono<Map<String, String>> createClient(Map<String, String> map) {
        LOG.info("create client");

        LOG.info("calling auth-server create client endpoint {}", clientsEndpoint);

        WebClient.ResponseSpec responseSpec = webClientBuilder.build().post().uri(clientsEndpoint)
                .bodyValue(map).retrieve();

        return responseSpec.bodyToMono(new ParameterizedTypeReference<Map<String, String>>(){}).map(responseMap-> {
            LOG.info("got back response from auth-server create client call: {}", map);
            return responseMap;
        }).onErrorResume(throwable -> {
            String stringBuilder = "auth-server create client failed: " +
                    throwable.getMessage();
            LOG.error(stringBuilder);
            return Mono.just(Map.of("error", stringBuilder));
        });
    }```