# My Authorization Server

I started building OAuth2 Client management using the Spring Authorization Server while I was laid-off in March 2024.  I had plenty of time and wanted to put a front-end to manage OAuth2 Client and for setting up user and assoiciated roles with an organization.  

There is a great resource on how to build on [Spring Authorization Server](https://docs.spring.io/spring-authorization-server/reference/index.html).  I took those information to create a Spring Boot application that can do authentication using call out to microservices.  Anyways, in short I was able to build a prototype application for managing OAuth2 clients and users.  My application is available at [authorization.sonam.cloud](https://authorization.sonam.cloud).  There is also a corresponding [authzmanager](https://authzmanager.sonam.cloud) webapplication for managing client, user, roles and organizations.

# How to create OAuth2 client on my authorization server
I have put together few pages on how to [signup a user](../signup/README.md), [create a OAuth2 Client](../create-oauth2-app/README.md) with organization and roles.  I also have a [Spring Boot app](../java-oauth2-app/README.md) that uses the user to sign-in and retrieve the roles.
