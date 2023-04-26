# How to map Webclient response to a Page type in Spring based application

This page is about how to retrieve a Page of response from a webservice call in a Spring Webflux project.

In this example, I have a Rest endpoint at `/applications/"+applicationId+"/users`  that returns a page of users that is associated to this application based on this applicationId. This example comes from [applications](https://github.com/sonamsamdupkhangsar/application-rest-service/blob/210b3a45c71f5b27ccc5f450cd9db7d8f73bd13c/src/test/java/me/sonam/application/ApplicationRestServiceTest.java#L149) example.

So the following is my test api call used in my integration test case:
```
EntityExchangeResult<RestPage> usersResult = webTestClient.get().uri("/applications/"+applicationId+"/users")
                .headers(addJwt(jwt))
                .exchange().expectStatus().isOk().expectBody(RestPage.class).returnResult();
```

I am using the following defined RestPage that is defined as a class within this file:

```
@JsonIgnoreProperties(ignoreUnknown = true, value = {"pageable"})
class RestPage<T> extends PageImpl<T> {
    @JsonCreator(mode = JsonCreator.Mode.PROPERTIES)
    public RestPage(@JsonProperty("content") List<T> content,
                    @JsonProperty("number") int page,
                    @JsonProperty("size") int size,
                    @JsonProperty("totalElements") long total,
                    @JsonProperty("numberOfElements") int numberOfElements,
                    @JsonProperty("pageNumber") int pageNumber
    ) {
        super(content, PageRequest.of(page, size), total);
    }

    public RestPage(Page<T> page) {
        super(page.getContent(), page.getPageable(), page.getTotalElements());
    }
}
```

The `JsonProperty` annotation will depend on what elements are returned from the Rest implementation.  Therefore, the JsonProperty annotation may need to be adjusted accordingly.  

