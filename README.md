# Dockerfile HEALTHCHECK instruction in Docker 1.12

In docker 1.12 the HEALTHCHECK build instruction has been added to the Dockerfile which allows a healthcheck to be built into the image.  This is incredibly useful.

Full details can be found here : https://github.com/docker/docker/pull/23218/commits/b6c7becbfe1d76b1250f6d8e991e645e13808a9c

## Background

When starting a container there may be a delay between the container starting and a service becoming available.  This is common with containers running webservers or databases.  The container has started, the service has started but it still isn't accessible.  We normally work around this with some bash in the container looping until the service is accessible. Or we use a script outside of the container which loops until it can access the container service.

## Benefits

Adding a HEALTHCHECK instruction to the Dockerfile has lots of benefits. Here are 4 :

1. It allows to you define what "health" is
2. A user can see the health requirement in the Dockerfile 
3. If a container is not behaving as expected, exec in and manually run the healthcheck
4. Health status is reported in docker ps

## Example Dockerfile

```
FROM alpine:3.3

MAINTAINER tomwillfixit 

RUN apk update && apk add curl && rm -rf /var/cache/apk/*

COPY helloworld.bin / 

EXPOSE 80

HEALTHCHECK --interval=5s --timeout=3s --retries=3 \
      CMD curl -f http://localhost:80 || exit 1

ENTRYPOINT ["/helloworld.bin"]

``` 

## What do the options mean?

According to the pull request :
```
The options that can appear before `CMD` are:

* `--interval=DURATION` (default: `30s`)
* `--timeout=DURATION` (default: `30s`)
* `--retries=N` (default: `1`)

The health check will first run **interval** seconds after the container is
started, and then again **interval** seconds after each previous check completes.

If a single run of the check takes longer than **timeout** seconds then the check
is considered to have failed.

It takes **retries** consecutive failures of the health check for the container
to be considered `unhealthy`.
``` 

# Try it

## Build container image
```
docker build -t helloworld:healthcheck .
```

## Start helloworld container
```
container_id=$(docker run -d -P helloworld:healthcheck)
```

## Check container health
```
docker inspect --format '{{ .State.Health.Status }}' ${container_id}

```

