# Docker images for IBAMR

Dockerfiles for working with IBAMR and its dependencies. Compiled docker images
are available here:

https://hub.docker.com/r/wellsd2/ibamr/tags

# Repository Layout

- `dependencies/` contains Dockerfiles for building all of IBAMR's dependencies in
  several different configurations. These are used in IBAMR's CI scripts.
- `ibamr/` contains Dockerfiles for building both IBAMR and its dependencies, as
  well as some downstream projects (fiddle).

## Submitting your own images to docker hub

If you want to build your own images, do something like

    docker login
    docker build -t ibamr-0.10.1-deps .
    docker tag ibamr-0.10.1-bundled-deps wellsd2/ibamr:ibamr-0.10.1-bundled-deps
    docker image push wellsd2/ibamr:ibamr:ibamr-0.10.1-bundled-deps
