# My Authorization Server

I started building OAuth2 Client management using [Spring Authorization Server](https://docs.spring.io/spring-authorization-server/reference/index.html) while I was impacted with a rif from my previous employer in March of 2024.  I wanted to use that time to put a front-end to manage OAuth2 Client and for setting up user and assoiciated roles with an organization.  

I chose Spring Authorization Server because I enjoy programming in Java and using Spring Framework.  There is also a good support on StackOverflow that can help answer questions when running into difficulty with implementations.

My application is available at [authorization.sonam.cloud/issuer](https://authorization.sonam.cloud/issuer).  There is also a corresponding [authzmanager](https://authzmanager.sonam.cloud) webapplication for managing client, user, roles and organizations.

# How to pages
I have put together few pages on how to [signup a user](../signup/README.md), [create a OAuth2 Client](../create-oauth2-app/README.md) with organization and roles.  I have also put a [Spring Boot app](../java-oauth2-app/README.md) that uses the user to sign-in and retrieve the roles.
