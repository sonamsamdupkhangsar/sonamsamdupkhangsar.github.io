# Building custom yaml properties for Spring application
This document is about building a custom yaml properties to be used in a Spring Java based application.


## Yaml property
I find that it is best described by showing an example.   The following is a custom yaml property that I need for building a parent-child property relationship that is injected as a property object in a class.  This example comes from my [jwt-validator](https://github.com/sonamsamdupkhangsar/jwt-validator/blob/0326c2fd1e22645c2c051fca6f504aaad0072eba/src/test/resources/application.yml#L76) project.



```
jwtrequest:
  - in: /api/health/passheader
    out: /api/health/jwtreceiver
    jwt: request
  - in: /api/health/passheader
    out: /api/health/liveness
    jwt: forward
  - in: /api/health/forwardtoken
    out: /api/health/jwtreceiver
    jwt: forward
```


In the example shown above, my root property is `jwt-request` which has 3 child properties `in`, `out` and `jwt`.  For some context on the usage of this property I use the `in` and `out` property to map in my `jwt-validator` project to determine whether I need to generate or forward a `jwt` token during request matching of inbound and outbound http paths when a request is made internally by any application.  This is setup as a http filter.


## Yaml property to Java mapping
To build a custom Java object for the above yaml properties you can build a component object with the  `@ConfigurationProperties` annotation.  The following is 
[my example](https://github.com/sonamsamdupkhangsar/jwt-validator/blob/0326c2fd1e22645c2c051fca6f504aaad0072eba/src/main/java/me/sonam/security/util/JwtPath.java#L12) of how this is done:


```
@Component
@ConfigurationProperties
public class JwtPath {
    private List<JwtRequest> jwtrequest = new ArrayList();

    public List<JwtRequest> getJwtRequest() {
        return jwtrequest;
    }

    private Map<String, List> map = new HashMap<>();

    public JwtPath() {

    }

    public static class JwtRequest {
        private String in;
        private String out;
        private String jwt;
        public enum JwtOption {
            forward, request, doNothing
        }

        public JwtRequest() {
        }

        public String getIn() {
            return in;
        }

        public String getOut() {
            return out;
        }

        public void setIn(String in) {
            this.in = in;
        }

        public void setOut(String out) {
            this.out = out;
        }

        public String getJwt() {
            return jwt;
        }

        public void setJwt(String jwt) {
            this.jwt = jwt;
        }

        @Override
        public String toString() {
            return "JwtRequest{" +
                    "in='" + in + '\'' +
                    ", out='" + out + '\'' +
                    ", jwt='" + jwt + '\'' +
                    '}';
        }
    }
}
```


## Example usage
The following example shows how it is injected in a [test case](https://github.com/sonamsamdupkhangsar/jwt-validator/blob/0326c2fd1e22645c2c051fca6f504aaad0072eba/src/test/java/me/sonam/security/YamlConfigTest.java#L38):

```
    @Autowired
    private JwtPath jwtPath;

    @Test
    public void jwtPath() {
        LOG.info("jwt.path: {}", jwtPath.getJwtRequest().size());
        assertThat(jwtPath.getJwtRequest().size()).isEqualTo(3);

        LOG.info("jwtPath[0].toString: {}", jwtPath.getJwtRequest().get(0).toString());
        assertThat(jwtPath.getJwtRequest().get(0).getIn()).isEqualTo("/api/health/passheader");
        assertThat(jwtPath.getJwtRequest().get(0).getOut()).isEqualTo("/api/health/jwtreceiver");
        assertThat(jwtPath.getJwtRequest().get(0).getJwt()).isEqualTo("request");
...
    }
```

## test output

The test case output would look something like:
```
2023-04-03 09:28:25.874  INFO 55983 --- [           main] me.sonam.security.YamlConfigTest         : jwt.path: 3
2023-04-03 09:28:25.917  INFO 55983 --- [           main] me.sonam.security.YamlConfigTest         : jwtPath[0].toString: JwtRequest{in='/api/health/passheader', out='/api/health/jwtreceiver', jwt='request'}

```

## Conclusion
That is it on how to build a custom yaml property and map to a Java object in a Spring application.
