# Linting Docker Hub Images

Docker containers to play host to all the various linting tools.

This repository houses a set of Dockerfiles and associated scripts and configs,
for the following linters: 

- [CoffeeLint](http://www.coffeelint.org) -
[Docker Hub repo](https://hub.docker.com/r/cozero/linter-coffeelint/)
- [HAML-Lint](https://github.com/brigade/haml-lint) -
[Docker Hub repo](https://hub.docker.com/r/cozero/linter-haml-lint/)
- [RuboCop](http://batsov.com/rubocop/) -
[Docker Hub repo](https://hub.docker.com/r/cozero/linter-rubocop/)
- [Sass Lint](https://www.npmjs.com/package/sass-lint) -
[Docker Hub repo](https://hub.docker.com/r/cozero/linter-sass-lint/)

Each linter's Dockerfile is housed, along with some other resources (e.g. entry
point script, default config) in its own directory within this repository.
From these directories, Docker images are built and uploaded to Docker Hub.
(TODO: links to images on Docker Hub.)

## Requirements

To build the images, all you need is the
[Docker client](https://store.docker.com/search?type=edition&offering=community)
appropriate to your operating system.

To run the tests, you'll need Google's
[Container Structure Tests framework](https://github.com/GoogleCloudPlatform/container-structure-test),
which is available either as a binary executable (for Linux only) or as a
Docker image (for all platforms).

## Building the images

The process is the same for each of the linters. Using CoffeeLint as an example:

```
cd coffeelint 
docker build --rm -t cozero/linter-coffeelint .
```

## Running the tests 

Again, each linter can be tested in a similar way. You'll need to
[build the images](#building-the-images) before you can test them.

Once you've built the images, here are the steps, using CoffeeLint as an
example.

### On Linux

If you are on Linux, first ensure that you have installed the Container
Structure Tests binary to a location on your $PATH. Then:

```
cd coffeelint
structure-test -test.v -image cozero/linter-coffeelint test-config.yaml
```

### Non Linux

If you are not running Linux, the command to run is a little more complicated,
as you have to use the Container Structure Tests container. As this is
Docker-in-Docker, you'll need to ensure that the container has access to your
Docker client so it can run the linter image. These commands work on a Mac
(note that this has not been tested on Windows, and Linux users should use the
[Linux instructions](#on-linux) above):

```
cd coffeelint
docker run -v `pwd`:/test/ -v /var/run/docker.sock:/var/run/docker.sock \
  gcr.io/gcp-runtimes/container-structure-test -test.v \
  -image cozero/linter-coffeelint /test/test-config.yaml
```

### CI/CD

This is currently TODO. I haven't yet been able to get the tests to run on 
a BuildKite agent. See (TODO section)[#todo] below.

## Using the linters 

To run a linter on your project, you'll first need to
(build the linter image)[#building-the-images] and then, from your project
root:

```
docker run -v `pwd`:/app cozero/linter-coffeelint
```

Note that all linters expect the project code to be linted to be available
at the path `/app` (as per command above).

## Deployment

Once the images are built, they can be deployed to Docker Hub using the 
Docker CLI's `docker push`
(command)[https://ropenscilabs.github.io/r-docker-tutorial/04-Dockerhub.html].

## TODO

### Set up CI/CD pipeline

Ideally, on every `git push` to the repo, the pipeline would build the images
and run the tests against them. Additionally, on the master branch - if the
tests pass - the pipeline should `docker push` the images to Docker Hub.

### Improve tests

Currently the tests only check two things: that the relevant linter is
available in each linter container, and that it is at the expected version.

Some sense of the actual usefulness of the linters might be a good thing to
test, too. Maybe the tests could also run the images against an example
app that should fail all the linters in certain ways. However, the
disadvantage of such an integration test is that we could end up testing
the linters themselves - a waste of time because each linter has its own 
test suite. At least _some_ indication of the utility of the images would
be useful, though.
