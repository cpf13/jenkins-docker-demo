# jenkins-docker-demo

## Building a container

You know that a container image is a single file that contains everything the container needs to run, but how do you build an image in the first place? Well, to do that, you use the `docker image build` command, which takes as input a special text file called a _Dockerfile_. The Dockerfile specifies exactly what needs to go into the container image.

One of the key benefits of containers is the ability to build on existing images to create new images. For example, you could take a container image containing the complete Ubuntu operating system, add a single file to it, and the result will be a new image.

In general, a Dockerfile has instructions for taking a starting image (a so-called _base image_), transforming it in some way, and saving the result as a new image.

## Understanding Dockerfiles

Let's see the [Dockerfile](Dockerfile) for our demo application:

```dockerfile
FROM golang:1.14-alpine AS build

WORKDIR /src/
COPY main.go go.* /src/
RUN CGO_ENABLED=0 go build -o /bin/demo

FROM scratch
COPY --from=build /bin/demo /bin/demo
ENTRYPOINT ["/bin/demo"]
```

The exact details of how this works don't matter for now, but it uses a fairly standard build process for Go containers called _multi-stage builds_. The first stage starts from an official `golang` container image, which is just an operating system (in this case Alpine Linux) with the Go language environment installed. It runs the `go build` command to compile the `main.go` file we saw earlier.

The result of this is an executable binary file named `demo`. The second stage takes a completely empty container image (called a _scratch_ image, as in 'from scratch') and copies the `demo` binary into it.

## Running 'docker image build'

We've seen that the Dockerfile contains instructions for the `docker image build` tool to turn our Go source code into an executable container. Let's go ahead and try it. In this directory, run the following command:

```bash
docker image build -t myhello .
Sending build context to Docker daemon  4.096kB
Step 1/7 : FROM golang:1.14-alpine AS build
...
Successfully built eeb7d1c2e2b7
Successfully tagged myhello:latest
```

Congratulations, you just built your first container! You can see from the output that Docker performs each of the actions in the Dockerfile in sequence on the newly-formed container, resulting in an image that's ready to use.

## Naming your images

When you build an image, by default it just gets a hexadecimal ID, which you can use to refer to it later (for example, to run it). These IDs aren't particularly memorable or easy to type, so Docker allows you to give the image a human-readable name, using the `-t` switch to `docker image build`. In the previous example you named the image `myhello`, so you should be able to use that name to run the image now.

Let's see if it works:

```
docker container run -p 9999:8080 myhello
```

You're now running your own copy of the demo application, and you can check it by browsing to the same URL as before:

http://localhost:9999/

You should see `Hello, 世界`. When you're done running this image, press Ctrl-C to stop the `docker container run` command.