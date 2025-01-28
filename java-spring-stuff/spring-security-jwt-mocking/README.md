# How to test forwarding of Jwt access-token to other services
This demo is about testing a Rest api that is secured using Spring Security.

The build configuration for this demo uses Gradle kotlin:
```
plugins {
    // Apply the java-library plugin for API and implementation separation.
    `java-library`
    id("org.springframework.boot") version "3.4.1"  // Replace with your desired version
    id("io.spring.dependency-management") version "1.1.6" // Dependency management plugin
    id("maven-publish")
}

dependencyManagement {
    imports {
        mavenBom ("org.springframework.cloud:spring-cloud-dependencies:2024.0.0")
    }
}

dependencies {
    // Use JUnit Jupiter for testing.
    testImplementation("org.junit.jupiter:junit-jupiter:5.9.3")

    testRuntimeOnly("org.junit.platform:junit-platform-launcher")
  
    implementation("org.springframework.boot:spring-boot-starter-security")
    testImplementation("org.springframework.security:spring-security-test")
//redacted other dependencies that are not needed
}
```

This demo comes from my [friendships-api](https://github.com/sonamsamdupkhangsar/friendships-api) repository.

I have some endpoint at path `/friendships` that is secured using Spring Security.  In order to access these endpoints I need to do some mocking of JWT tokents to access those endpoints. I also want to send some user attributes in the token like userId of UUID type.

I am going to pick my simplest endpoint which checks if the logged-in user is friends with the supplied userId with path `/friendships/{userId}`.  In this test case I use MockWebServer to mock responses for Rest callouts which is not used in this particular test.  The following is my Java code:


```
/**
 * this will test the end-to-end from the Router to business service to entity persistence using in-memory db.
 */
@EnableAutoConfiguration
@ExtendWith(SpringExtension.class)
@SpringBootTest(classes = {Application.class, TestConfig.class}, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ExtendWith(MockitoExtension.class)
public class FriendshipRouterIntegTest {
    private static final Logger LOG = LoggerFactory.getLogger(FriendshipRouterIntegTest.class);

    private static MockWebServer mockWebServer;
    @MockitoBean
    ReactiveJwtDecoder jwtDecoder;

    @Value("${friendshipEndpoint}")
    private String friendshipEndpoint;

    @Autowired
    private WebTestClient webTestClient;

    @Autowired
    ApplicationContext context;

    @Autowired
    private FriendshipRepository friendshipRepository;

    @org.junit.jupiter.api.BeforeEach
    public void setup() {
        this.webTestClient = WebTestClient
                .bindToApplicationContext(this.context)
                // add Spring Security test Support
                .apply(springSecurity())
                .configureClient()
                //   .filter(basicAuthentication("user", "password"))
                .build();
    }

    @BeforeAll
    static void setupMockWebServer() throws IOException {
        mockWebServer = new MockWebServer();
        mockWebServer.start();

        LOG.info("host: {}, port: {}", mockWebServer.getHostName(), mockWebServer.getPort());
    }

    @AfterAll
    public static void shutdownMockWebServer() throws IOException {
        LOG.info("shutdown and close mockWebServer");
        mockWebServer.shutdown();
        mockWebServer.close();
    }

    @DynamicPropertySource
    static void properties(DynamicPropertyRegistry r) throws IOException {
        r.add("friendshipEndpoint", () -> "http://localhost:"+ mockWebServer.getPort());
        r.add("userEndpoint", () -> "http://localhost:"+ mockWebServer.getPort());
        r.add("notificationEndpoint", ()->"http://localhost:"+mockWebServer.getPort());
    }

  // this isFriends() just setups the test case.  The actual Rest call is "private void isFriends(Jwt jwt, UUID userId, boolean isFriends)"
 @Test
    public void isFriends() throws InterruptedException {
        LOG.info("isFriends endpoint test");

        //for this set friendId to hardcoded value so that we can pass this friendId into jwt
        //which will accept the friendship, user cannot accept only the friend in the friendship can
        UUID userId = UUID.fromString("5d8de63a-0b45-4c33-b9eb-d7fb8d662107");
        String authenticationId = "dave";
        Jwt jwt = jwt(authenticationId, userId);
        when(this.jwtDecoder.decode(anyString())).thenReturn(Mono.just(jwt));

        UUID friendId = UUID.randomUUID();
        LOG.info("friendId: {}",friendId);

        Friendship friendship = new Friendship(LocalDateTime.now(), LocalDateTime.now(), userId, friendId, true);
        friendshipRepository.save(friendship).subscribe();

        LOG.info("verify user {} is friend with {}", userId, friendId);
        isFriends(jwt, friendId, true);

        LOG.info("delete all friendships before testing another one");
        friendshipRepository.deleteAll().subscribe();

        // swap userId and friendId
        friendId = UUID.fromString("5d8de63a-0b45-4c33-b9eb-d7fb8d662107");
        authenticationId = "dave";
        jwt = jwt(authenticationId, friendId);
        when(this.jwtDecoder.decode(anyString())).thenReturn(Mono.just(jwt));

        userId = UUID.randomUUID();
        LOG.info("friendId: {}",friendId);

        friendship = new Friendship(LocalDateTime.now(), LocalDateTime.now(), friendId, userId, true);
        friendshipRepository.save(friendship).subscribe();

        LOG.info("verify user {} is friend with {}", userId, friendId);
        isFriends(jwt, userId, true);

        LOG.info("delete all friendships before testing another one");
        friendshipRepository.deleteAll().subscribe();

        friendship = new Friendship(LocalDateTime.now(), LocalDateTime.now(), friendId, userId, false);
        friendshipRepository.save(friendship).subscribe();

        LOG.info("verify user {} is friend with {}", userId, friendId);
        isFriends(jwt, userId, false);
    }

    private void isFriends(Jwt jwt, UUID userId, boolean isFriends) {
        LOG.info("check user is friends userId: {}", userId);

        String endpoint = "/friendships/{userId}".replace("{userId}", userId.toString());

        EntityExchangeResult<String> entityExchangeResult = webTestClient.
                mutateWith(mockJwt().jwt(jwt)).get().uri(endpoint)
                .headers(addJwt(jwt))
                .accept(MediaType.APPLICATION_NDJSON)
                .exchange().expectStatus().isOk().expectBody(String.class)
                .returnResult();

        LOG.info("response is {}", entityExchangeResult.getResponseBody());

        assertThat(entityExchangeResult.getResponseBody()).isEqualTo("{\"message\":"+isFriends+"}");
    }

    private Jwt jwt(String subjectName, UUID userId) {
        return new Jwt("token", null, null,
                Map.of("alg", "none"), Map.of("sub", subjectName, "userId", userId.toString()));
    }

    private Consumer<HttpHeaders> addJwt(Jwt jwt) {
        return headers -> headers.setBearerAuth(jwt.getTokenValue());
    }
```


# Explanation of the test code
The setup block
```
@org.junit.jupiter.api.BeforeEach 
   public void setup() {
``` 

is necessary for the `webTestClient.mutateWith(mockJwt().jwt(jwt)).get()` call to work otherwise you can get http client filter errors or null pointers.


The actual Rest call out is this block 
```
private void isFriends(Jwt jwt, UUID userId, boolean isFriends) {
        LOG.info("check user is friends userId: {}", userId);

        String endpoint = "/friendships/{userId}".replace("{userId}", userId.toString());

        EntityExchangeResult<String> entityExchangeResult = webTestClient.
                mutateWith(mockJwt().jwt(jwt)).get().uri(endpoint)
                .headers(addJwt(jwt))
                .accept(MediaType.APPLICATION_NDJSON)
                .exchange().expectStatus().isOk().expectBody(String.class)
                .returnResult();
```                

When the test executes, it passes the Jwt to the service and the service can now access the userId attribute for the logged-in user.  I have a `getLoggedInUserId()` method in my business service to extract userId that is in the Jwt token that is this:
```
 public Mono<UUID> getLoggedInUserId() {
        return ReactiveSecurityContextHolder.getContext().flatMap(securityContext -> {
            org.springframework.security.core.Authentication authentication = securityContext.getAuthentication();
            Jwt jwt = (Jwt) authentication.getPrincipal();
            String userIdString = jwt.getClaim("userId");
            LOG.debug("claims: {}, jwt: {}, security context userId: {}", jwt.getClaims(), jwt,
                    userIdString);

            UUID userId = UUID.fromString(userIdString);

            return Mono.just(userId);
        });
    }

```

The following is a sample output when the test case is run:
```
2025-01-28T08:46:07.174-07:00  INFO 64703 --- [    Test worker] m.s.f.FriendshipRouterIntegTest          : check user is friends userId: 6e97eb8c-f0af-408e-9802-d5026485083a
2025-01-28T08:46:07.260-07:00  INFO 64703 --- [     parallel-3] me.sonam.friendships.FriendshipService   : isFriends with 6e97eb8c-f0af-408e-9802-d5026485083a
2025-01-28T08:46:07.261-07:00 DEBUG 64703 --- [     parallel-3] me.sonam.friendships.FriendshipService   : claims: {userId=5d8de63a-0b45-4c33-b9eb-d7fb8d662107, sub=dave}, jwt: org.springframework.security.oauth2.jwt.Jwt@bbd01fb9, security context userId: 5d8de63a-0b45-4c33-b9eb-d7fb8d662107
2025-01-28T08:46:07.346-07:00  INFO 64703 --- [    Test worker] m.s.f.FriendshipRouterIntegTest          : response is {"message":true}

```