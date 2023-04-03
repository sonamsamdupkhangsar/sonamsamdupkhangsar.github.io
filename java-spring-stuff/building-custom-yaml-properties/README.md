# Building custom yaml properties for Spring Framework based application
This document is about building a custom yaml properties to be used in a Spring Java based application.


I find that it is best described by showing an example.   The following is a custom yaml property that I need to build for building a parent-child property relationship that is injected as a property object in a class.  This example comes from my [jwt-validator](https://github.com/sonamsamdupkhangsar/jwt-validator/blob/0326c2fd1e22645c2c051fca6f504aaad0072eba/src/test/resources/application.yml#L76) project.



```
jwtrequest:
  - in: /api/health/passheader
    out: /jwts/accesstoken
    jwt: request
  - in: /api/health/passheader
    out: /api/health/jwtreceiver
    jwt: forward
```


In the example shown above, my root property is `jwt-request` which has 3 child properties `in`, `out` and `jwt`.  For some context on the usage of this property I use the `in` and `out` property to map in my `jwt-validator` project to determine whether I need to generate a `jwt` token request for matching paths when a request is made internally by any application.  


<br/>
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


That is it on how to build a custom yaml property and map to a Java object in a Spring application.
