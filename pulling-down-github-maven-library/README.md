# How to use maven library from github in your maven project?
This is about how to pull down maven dependency from a github repository and use in one of your other maven project. 

Table Of Contents

1. [Include Jar in Maven Project](#include-jar-in-maven-project)
2. [Customize settings.xml](#customize-settingsxml)
3. [Building maven project](#building-maven-project)
4. [Integrating into Github Action with Dockerfile](#integrating-into-github-action-with-dockerfile)

## Include Jar in Maven Project
In order to use the maven artifact you will first include it in the `pom.xml` file.  For this article purposes, I am going to use my github maven project [jwt-validator](https://github.com/sonamsamdupkhangsar/jwt-validator) as a dependency to be included in another project.  

This dependency library is for validating jwt token which can be included in a pom file as:

```
<dependency>
  <groupId>me.sonam</groupId>
  <artifactId>jwt-validator</artifactId>
  <version>1.0-SNAPSHOT</version>
</dependency>
```

Then you have to follow the next section [Customize settings.xml](#customize-settingsxml) section below to update your settings file.
<br/>


## Customize settings.xml
To pull down a github repository maven artifact you will have to update your maven's `settings.xml` to reference the github repository where the artifiact is and also have a Personal Access Token.

You can setup a personal access token by visiting your Github profile, then to `Developer settings` to generate a new token.  You can use Personal access tokens (classic).  There is also a newer way of creating `Fine-grained tokens` but I am using classic token for this example.


The following is a settings.xml file for downloading [jwt-validator](https://github.com/sonamsamdupkhangsar/jwt-validator) maven artifact from my Github repository to be used in another project like my [application-rest-service](https://github.com/sonamsamdupkhangsar/application-rest-service) for validating user request based on a jwt token:

```
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <activeProfiles>
        <activeProfile>github</activeProfile>
    </activeProfiles>
    <profiles>
        <profile>
            <id>github</id>
            <repositories>
                <repository>
                    <id>central</id>
                    <url>https://repo1.maven.org/maven2</url>
                </repository>
                <repository>
                    <id>github</id>
                    <url>https://maven.pkg.github.com/sonamsamdupkhangsar/jwt-validator</url>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </repository>
            </repositories>
        </profile>
    </profiles>

    <servers>
        <server>
            <id>github</id>
            <username>sonamsamdupkhangsar</username>
            <password>${PERSONAL_ACCESS_TOKEN}</password>
        </server>
    </servers>
</settings>
```

In the above settings.xml file I am referencing my repository `https://maven.pkg.github.com/sonamsamdupkhangsar/jwt-validator` where the artifact is hosted.  See this [documentation] (https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-apache-maven-registry) for more on maven repository.

Then I also specify my PERSONAL_ACCESS_TOKEN as a variable that can be injected from the environment.

To build the maven project follow the [Building maven project](#building-maven-project) below.
<br/>

## Building maven project
After you have included the maven library in your pom and customized the `settings.xml` you can download the library and build the project as `mvn clean install` for example.  

However, if you need to have a custom `settings.xml` then you can specify the settings file as argument to the maven build command as:

```
mvn -s settings.xml clean package
```

This assumes you have a environment variable already exported for PERSONAL_ACCESS_TOKEN.  

You can export the environment variable as :
```
export PERSONAL_ACCESS_TOKEN=<your pat value>
```

If you want to integrate the above settings file in a Github action pipeline follow [Integrating into Github Action with Dockerfile](#integrating-into-github-action-with-dockerfile) below.
<br/>


## Integrating into Github Action with Dockerfile
For this article, I am building my project using Github action pipeline.  I am using `docker/build-push-action@v3` action for building my docker image as follows:

```
- name: Build Docker image and push to ghcr.io registry
  id: docke/_build
  uses: docker/build-push-action@v3
  with:
    context: .
    push: true
    tags: ghcr.io/sonamsamdupkhangsar/jwt-rest-service:latest
    secrets: |
      GIT_AUTH_TOKEN=${{ secrets.PERSONAL_ACCESS_TOKEN }}
      PERSONAL_ACCESS_TOKEN=${{ secrets.PERSONAL_ACCESS_TOKEN }}
```
I specify my action using `uses` line. I specify the repository tags with `tags`. I also specify my Github secret with `secrets` for PERSONAL_ACCESS_TOKEN to be mounted as docker secret volume.

The above Github action will mount the secret value in a docker secret volume.  You can then import the secret by creating a environment variable in your Dockerfile `RUN` command using the exec shell instruction.  

The docker secret can be accessible to the maven instruction after extracting from docker secret volume and creating similar named variable in the `RUN` command below.  

The following is the dockerfile for building the maven package with the secret:

```
FROM maven:3-openjdk-17-slim as build

WORKDIR /app

#copy pom and settings.xml too
COPY pom.xml settings.xml ./

COPY src ./src

# use exec shell form to access secret variable as exported env variable
RUN --mount=type=secret,id=PERSONAL_ACCESS_TOKEN \
   export PERSONAL_ACCESS_TOKEN=$(cat /run/secrets/PERSONAL_ACCESS_TOKEN) && \
   mvn -s settings.xml clean install
```

You should now be able to build the Dockerfile with your custom `settings.xml` file.  For reference on how this is all working you can checkout the https://github.com/sonamsamdupkhangsar/jwt-rest-service