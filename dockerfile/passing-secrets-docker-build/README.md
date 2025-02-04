# About
This README is about passing variables into a dockerfile instruction and which could be set as environment variable to be used in the gradle or maven build.

# How to use secrets in a local build
We generally have projects that depend on external dependencies.  For example, we might pull down maven dependencies from external github repository.  In order to pull a github artifact you need to submit a `username` and a `personal_access_token`.  

For example, you may have a Docker instruction as following to build a gradle project in a dockerfile:

```RUN --mount=type=secret,id=USERNAME --mount=type=secret,id=PERSONAL_ACCESS_TOKEN --mount=type=cache,target=/root/.gradle\
    export USERNAME=$(cat /run/secrets/USERNAME)\
    export PERSONAL_ACCESS_TOKEN=$(cat /run/secrets/PERSONAL_ACCESS_TOKEN) &&\
     ./gradlew clean build
```

In order to pass the `USERNAME` and the `PERSONAL_ACCESS_TOKEN` you need to pass those variables from a command line as following:

 ```
 export USERNAME=<username>
 export PERSONAL_ACCESS_TOKEN=<pat>

docker build --secret id=USERNAME,env=USERNAME --secret id=PERSONAL_ACCESS_TOKEN,env=PERSONAL_ACCESS_TOKEN -t mydockerimage .
```

In the above example, I am assuming that you want to pass the secret using a environment variable by exporting it first.  

However, you can also pass them using a file by passing a `src`.  In this example of using a file, you need to create a `USERNAME` and `PERSONAL_ACCESS_TOKEN` file and put their approriate content in them.  The following is the docker build command line passing the `src` of the file name:

```
docker build --secret id=USERNAME,src=USERNAME --secret id=PERSONAL_ACCESS_TOKEN,src=PERSONAL_ACCESS_TOKEN -t mydockerimage .
```

