# How to test forwarding of Jwt access-token to other services
This demo is about accessing a secured api that sits behind a secured service like OAuth2. This demo assumes the use
of Spring security library `org.springframework.security:spring-security-test`

I have a `/api/health/passheader` endpoint.  I also have another endpoint that receives the jwt access token at `/api/health/jwtreceiver`.
As the name implies on the endpoints one gets a jwt header and forwards to another endpoint.

This is about testing so I will write a test case that shows this.  I have written a following test case:

```
@EnableAutoConfiguration
@ExtendWith(SpringExtension.class)
@SpringBootTest(classes = {Application.class}, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ExtendWith(MockitoExtension.class)
public class JwtHeaderPassIntegTest {
    private static final Logger LOG = LoggerFactory.getLogger(JwtHeaderPassIntegTest.class);

    @Autowired
    private WebTestClient client;

    @MockBean
    ReactiveJwtDecoder jwtDecoder;
    private static MockWebServer mockWebServer;

    @Autowired
    ApplicationContext context;

    // For `client.mutateWith(mockJwt().jwt(jwt)).` to work you must set this up otherwise you
    // will get a binding http client filter error
    @org.junit.jupiter.api.BeforeEach
    public void setup() {
        this.client = WebTestClient
                .bindToApplicationContext(this.context)
                // add Spring Security test Support
                .apply(springSecurity())
                .configureClient()
                .build();
    }
    
    /**
     * this method tests http delete method overridden for calling '/api/health/passheader -> '/api/health/jwtreceiver'
     * @throws InterruptedException
     */

    @Test
    public void passExistingHeaderJwtForDelete() throws InterruptedException {
        LOG.info("readiness delete requires jwt, should get bad request");

        UUID userId = UUID.randomUUID();
        final String authenticationId = "dave";
        Jwt jwt = jwt(authenticationId, userId);
        Mockito.when(this.jwtDecoder.decode(ArgumentMatchers.anyString())).thenReturn(Mono.just(jwt));

        final String jwtString= "eyJraWQiOiJlOGQ3MjIzMC1iMDgwLTRhZjEtODFkOC0zMzE3NmNhMTM5ODIiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiI3NzI1ZjZmZC1kMzk2LTQwYWYtOTg4Ni1jYTg4YzZlOGZjZDgiLCJhdWQiOiI3NzI1ZjZmZC1kMzk2LTQwYWYtOTg4Ni1jYTg4YzZlOGZjZDgiLCJuYmYiOjE3MTQ3NTY2ODIsImlzcyI6Imh0dHA6Ly9teS1zZXJ2ZXI6OTAwMSIsImV4cCI6MTcxNDc1Njk4MiwiaWF0IjoxNzE0NzU2NjgyLCJqdGkiOiI0NDBlZDY0My00MzdkLTRjOTMtYTZkMi1jNzYxNjFlNDRlZjUifQ.fjqgoczZbbmcnvYpVN4yakpbplp7EkDyxslvar5nXBFa6mgIFcZa29fwIKfcie3oUMQ8MDWxayak5PZ_QIuHwTvKSWHs0WL91ljf-GT1sPi1b4gDKf0rJOwi0ClcoTCRIx9-WGR6t2BBR1Rk6RGF2MW7xKw8M-RMac2A2mPEPJqoh4Pky1KgxhZpEXixegpAdQIvBgc0KBZeQme-ZzTYugB8EPUmGpMlfd-zX_vcR1ijxi8e-LRRJMqmGkc9GXfrH7MOKNQ_nu6pc6Gish2v_iuUEcpPHXrfqzGb9IHCLvfuLSaTDcYKYjQaEUAp-1uDW8-5posjiUV2eBiU48ajYg";

        final String jwtReceiver = " {\"message\":\"jwt received endpoint\"}";
        mockWebServer.enqueue(new MockResponse().setHeader("Content-Type", "application/json").setResponseCode(200).setBody(jwtReceiver));//"Account created successfully.  Check email for activating account"));

        LOG.info("call passheader endpoint");
        client.mutateWith(mockJwt().jwt(jwt)).delete().uri("/api/health/passheader")
            //    .headers(addJwt(jwt))
                .headers(httpHeaders -> httpHeaders.setBearerAuth(jwtString))
                .exchange().expectStatus().isOk();

        RecordedRequest recordedRequest = mockWebServer.takeRequest();

        LOG.info("should be acesstoken path for recordedRequest: {}", recordedRequest.getPath());
        AssertionsForClassTypes.assertThat(recordedRequest.getPath()).startsWith("/api/health/jwtreceiver");
        AssertionsForClassTypes.assertThat(recordedRequest.getMethod()).isEqualTo("GET");
    }

```

This test cases project uses a token-filter library I wrote for forwarding tokens using configuration based on path mappings.
  The path mapping yaml allows to write a configuration whether you want to forward tokens, 
  generate new access-token using client-credential flow or to not forward any tokens to downstream api.   The yaml looks like:

```
requestFilters:
  - in: /api/health/passheader
    out: /api/health/jwtreceiver
    httpMethods: delete
    accessToken:
      option: forward
  - in: /api/health/passheader
    out: /api/health/jwtreceiver
    httpMethods: get, post, put
    accessToken:
      option: request
      scopes: message.read message.write
      base64EncodedClientIdSecret: b2F1dGgtY2xpZW50Om9hdXRoLXNlY3JldA==
    ```
So in this case I have 2 mappings for the same inbound and outbound paths where for 1 mapping I forward the access-token for delete operation.  For `get, post, put` http methods I generate new tokens using client credential flow.

When the test case runs it will forward the jwt for the delete operation.  Here is the output log:

```
2024-07-21T13:00:49.159-06:00  INFO 2351 --- [    Test worker] m.sonam.security.JwtHeaderPassIntegTest  : readiness delete requires jwt, should get bad request
2024-07-21T13:00:49.177-06:00  INFO 2351 --- [    Test worker] m.sonam.security.JwtHeaderPassIntegTest  : call passheader endpoint
2024-07-21T13:00:49.274-06:00 DEBUG 2351 --- [     parallel-2] m.s.s.h.ReactiveRequestContextFilter     : request: /api/health/passheader
2024-07-21T13:00:49.285-06:00 DEBUG 2351 --- [     parallel-2] me.sonam.security.EndpointHandler        : pass jwt header to receiveJwtHeader endpoint
2024-07-21T13:00:49.285-06:00  INFO 2351 --- [     parallel-2] me.sonam.security.EndpointHandler        : call endpoint: http://localhost:56944/api/health/jwtreceiver
2024-07-21T13:00:49.297-06:00  INFO 2351 --- [     parallel-2] m.s.s.h.ReactiveRequestContextHolder     : serverHttpRequest: /api/health/passheader
2024-07-21T13:00:49.297-06:00  INFO 2351 --- [     parallel-2] m.s.s.h.ReactiveRequestContextHolder     : request path: /api/health/jwtreceiver, accessTokenPath: /oauth2/token
2024-07-21T13:00:49.297-06:00  INFO 2351 --- [     parallel-2] m.s.s.h.ReactiveRequestContextHolder     : in path: /api/health/passheader, outbound path: /api/health/jwtreceiver
2024-07-21T13:00:49.298-06:00  INFO 2351 --- [     parallel-2] m.s.s.h.ReactiveRequestContextHolder     : httpMethods: [delete]
2024-07-21T13:00:49.298-06:00  INFO 2351 --- [     parallel-2] m.s.s.h.ReactiveRequestContextHolder     : request.httpMethod: DELETE
2024-07-21T13:00:49.298-06:00  INFO 2351 --- [     parallel-2] m.s.s.h.ReactiveRequestContextHolder     : request.method DELETE matched with provided httpMethod delete
2024-07-21T13:00:49.298-06:00 DEBUG 2351 --- [     parallel-2] m.s.s.h.ReactiveRequestContextHolder     : inPath: /api/health/passheader
2024-07-21T13:00:49.298-06:00  INFO 2351 --- [     parallel-2] m.s.s.h.ReactiveRequestContextHolder     : inbound request path /api/health/passheader matches with inPath parameter: /api/health/passheader
2024-07-21T13:00:49.298-06:00 DEBUG 2351 --- [     parallel-2] m.s.s.h.ReactiveRequestContextHolder     : check outbound path now
2024-07-21T13:00:49.299-06:00 DEBUG 2351 --- [     parallel-2] m.s.s.h.ReactiveRequestContextHolder     : outPath: /api/health/jwtreceiver
2024-07-21T13:00:49.299-06:00  INFO 2351 --- [     parallel-2] m.s.s.h.ReactiveRequestContextHolder     : outbound path /api/health/jwtreceiver matches with outbound parameter: /api/health/jwtreceiver
2024-07-21T13:00:49.301-06:00  INFO 2351 --- [     parallel-2] m.s.s.h.ReactiveRequestContextHolder     : accessTokenHeader: Bearer eyJraWQiOiJlOGQ3MjIzMC1iMDgwLTRhZjEtODFkOC0zMzE3NmNhMTM5ODIiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiI3NzI1ZjZmZC1kMzk2LTQwYWYtOTg4Ni1jYTg4YzZlOGZjZDgiLCJhdWQiOiI3NzI1ZjZmZC1kMzk2LTQwYWYtOTg4Ni1jYTg4YzZlOGZjZDgiLCJuYmYiOjE3MTQ3NTY2ODIsImlzcyI6Imh0dHA6Ly9teS1zZXJ2ZXI6OTAwMSIsImV4cCI6MTcxNDc1Njk4MiwiaWF0IjoxNzE0NzU2NjgyLCJqdGkiOiI0NDBlZDY0My00MzdkLTRjOTMtYTZkMi1jNzYxNjFlNDRlZjUifQ.fjqgoczZbbmcnvYpVN4yakpbplp7EkDyxslvar5nXBFa6mgIFcZa29fwIKfcie3oUMQ8MDWxayak5PZ_QIuHwTvKSWHs0WL91ljf-GT1sPi1b4gDKf0rJOwi0ClcoTCRIx9-WGR6t2BBR1Rk6RGF2MW7xKw8M-RMac2A2mPEPJqoh4Pky1KgxhZpEXixegpAdQIvBgc0KBZeQme-ZzTYugB8EPUmGpMlfd-zX_vcR1ijxi8e-LRRJMqmGkc9GXfrH7MOKNQ_nu6pc6Gish2v_iuUEcpPHXrfqzGb9IHCLvfuLSaTDcYKYjQaEUAp-1uDW8-5posjiUV2eBiU48ajYg
2024-07-21T13:00:49.301-06:00  INFO 2351 --- [     parallel-2] m.s.s.h.ReactiveRequestContextHolder     : pass inbound accessToken : eyJraWQiOiJlOGQ3MjIzMC1iMDgwLTRhZjEtODFkOC0zMzE3NmNhMTM5ODIiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiI3NzI1ZjZmZC1kMzk2LTQwYWYtOTg4Ni1jYTg4YzZlOGZjZDgiLCJhdWQiOiI3NzI1ZjZmZC1kMzk2LTQwYWYtOTg4Ni1jYTg4YzZlOGZjZDgiLCJuYmYiOjE3MTQ3NTY2ODIsImlzcyI6Imh0dHA6Ly9teS1zZXJ2ZXI6OTAwMSIsImV4cCI6MTcxNDc1Njk4MiwiaWF0IjoxNzE0NzU2NjgyLCJqdGkiOiI0NDBlZDY0My00MzdkLTRjOTMtYTZkMi1jNzYxNjFlNDRlZjUifQ.fjqgoczZbbmcnvYpVN4yakpbplp7EkDyxslvar5nXBFa6mgIFcZa29fwIKfcie3oUMQ8MDWxayak5PZ_QIuHwTvKSWHs0WL91ljf-GT1sPi1b4gDKf0rJOwi0ClcoTCRIx9-WGR6t2BBR1Rk6RGF2MW7xKw8M-RMac2A2mPEPJqoh4Pky1KgxhZpEXixegpAdQIvBgc0KBZeQme-ZzTYugB8EPUmGpMlfd-zX_vcR1ijxi8e-LRRJMqmGkc9GXfrH7MOKNQ_nu6pc6Gish2v_iuUEcpPHXrfqzGb9IHCLvfuLSaTDcYKYjQaEUAp-1uDW8-5posjiUV2eBiU48ajYg
2024-07-21T13:00:49.663-06:00  INFO 2351 --- [ctor-http-nio-3] me.sonam.security.EndpointHandler        : response for endpoint 'http://localhost:56944/api/health/jwtreceiver' is: {message=jwt received endpoint}
2024-07-21T13:00:49.664-06:00  INFO 2351 --- [ctor-http-nio-3] me.sonam.security.EndpointHandler        : got response message: jwt received endpoint
2024-07-21T13:00:49.699-06:00  INFO 2351 --- [    Test worker] m.sonam.security.JwtHeaderPassIntegTest  : should be acesstoken path for recordedRequest: /api/health/jwtreceiver

```



