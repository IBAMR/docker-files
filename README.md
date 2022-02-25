# Docker images for IBAMR

These images contain all of IBAMR's dependencies. Presently, we just have a
single dockerfile corresponding to Fedora 33.

## Submitting your own images to docker hub

If you want to build your own images, do something like

    docker login
    docker build -t ibamr-0.10.1-deps .
    docker tag ibamr-0.10.1-bundled-deps wellsd2/ibamr:ibamr-0.10.1-bundled-deps
    docker image push wellsd2/ibamr:ibamr:ibamr-0.10.1-bundled-deps
