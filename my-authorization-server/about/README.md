# My Authorization Server

I started building my OAuth2 Client management using [Spring Authorization Server](https://docs.spring.io/spring-authorization-server/reference/index.html) while I was impacted with a rif from my previous employer in March of 2024.  I wanted to use that time to put a front-end to manage OAuth2 Client and for setting up user and assoiciated roles with an organization.  

I chose Spring Authorization Server because I enjoy programming in Java and using Spring Framework.  There is also a good support on StackOverflow that can help answer questions when running into difficulty with implementations.

My application is available at [authorization.sonam.cloud/issuer](https://authorization.sonam.cloud/issuer).  There is also a corresponding [authzmanager](https://authzmanager.sonam.cloud) webapplication for managing client, user, roles and organizations.

# How to pages
I have put together few pages on how to [signup a user](../signup/README.md), [create a OAuth2 Client](../create-oauth2-app/README.md) with organization and roles.  I have also put a [Spring Boot app](../java-oauth2-app/README.md) that uses the user to sign-in and retrieve the roles.


# Using it
You are welcome to use my Authorization server for creating OAuth2 Client and signinup users.  This authorization server works only for a small use case and will need more extending.  

# Services
The following artifacts are used to manage my authorization server:
1. [My Authoriation Server or the issuer](https://github.com/sonamsamdupkhangsar/authorization) hosted at [authorization.sonam.cloud/issuer](https://authorization.sonam.cloud/issuer/).
2. [Authzmanager](https://github.com/sonamsamdupkhangsar/authzmanager) to manage Oauth clients/users/roles at [authzmanager.sonam.cloud](https://authzmanager.sonam.cloud).
3. [user-rest-servce](https://github.com/sonamsamdupkhangsar/user-rest-service) that manages user information and authentication.
4. [account-rest-service](https://github.com/sonamsamdupkhangsar/account-rest-service) to manage account creation, activation, emailing account activation link, validation email link.
5. [authentication-rest-service](https://github.com/sonamsamdupkhangsar/authentication-rest-service) is used by user-rest-service for account signup and authentication.
6. [role-rest-service](https://github.com/sonamsamdupkhangsar/role-rest-service) to  manage roles for user by organization.
7. [organization-rest-service](https://github.com/sonamsamdupkhangsar/organization-rest-service) for manage relationship between user, organization and Oauth client.
8. [email-rest-service](https://github.com/sonamsamdupkhangsar/email-rest-service) is used for sending email for account activation, password reset.