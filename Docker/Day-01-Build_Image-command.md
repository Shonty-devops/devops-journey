# Day 01 — Docker: Build an Image

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker Fundamentals — Building an image

---

## What are Images in Docker?

A Docker image is a read-only template used to create containers.

Think of a Docker image like a recipe for making a dish 🍔

It tells you:
what ingredients you need
how to prepare them
what steps to follow

👉 But the recipe itself is not the actual food
---

## Commands practised

| Command | What it does |
|---------|-------------|
| `vi Dockerfile`    | Opens a file editor, we define files and its contents |
| `FROM bash` | Using bash as a base |
| `RUN ping killercoda.com` | Defining what needs to be executed |
| `docker build -t pinger .`| Building an image named pinger using tag "-t" |
| `docker images ls`  | using ls to confirm how many images exist |
| `docker run --name my-ping pinger` | Docker run image, creates a container|

---




## Terminal observations
root@ubuntu:~$ vi Dockerfile
root@ubuntu:~$ docker build -t pinger .
DEPRECATED: The legacy builder is deprecated and will be removed in a future release.
            Install the buildx component to build images with BuildKit:
            https://docs.docker.com/go/buildx/

Sending build context to Docker daemon  23.55kB
Step 1/2 : FROM bash
latest: Pulling from library/bash
589002ba0eae: Pulling fs layer
bb56ad27587d: Pulling fs layer
ad56d8d99cca: Pulling fs layer
ad56d8d99cca: Verifying Checksum
ad56d8d99cca: Download complete
bb56ad27587d: Verifying Checksum
bb56ad27587d: Download complete
589002ba0eae: Verifying Checksum
589002ba0eae: Download complete
589002ba0eae: Pull complete
bb56ad27587d: Pull complete
ad56d8d99cca: Pull complete
Digest: sha256:e320b40b14abc61f053286705070342c46034a549a276b798e64541457c4af92
Status: Downloaded newer image for bash:latest
 ---> b66e847d14ac
Step 2/2 : CMD ["ping", "killerkoda.com"]
 ---> Running in 27ef9e277101
 ---> Removed intermediate container 27ef9e277101
 ---> 21490e4875b2
Successfully built 21490e4875b2
Successfully tagged pinger:latest
root@ubuntu:~$ docker run --name -my-ping pinger
docker: Error response from daemon: Invalid container name (-my-ping), only [a-zA-Z0-9][a-zA-Z0-9_.-] are allowed

Run 'docker run --help' for more information
root@ubuntu:~$ docker run --name my-ping pinger
PING killerkoda.com (162.255.119.208): 56 data bytes
64 bytes from 162.255.119.208: seq=0 ttl=43 time=164.746 ms
64 bytes from 162.255.119.208: seq=1 ttl=43 time=163.862 ms
64 bytes from 162.255.119.208: seq=2 ttl=43 time=164.182 ms
64 bytes from 162.255.119.208: seq=3 ttl=43 time=164.076 ms
64 bytes from 162.255.119.208: seq=4 ttl=43 time=163.936 ms
64 bytes from 162.255.119.208: seq=5 ttl=43 time=164.353 ms
64 bytes from 162.255.119.208: seq=6 ttl=43 time=164.448 ms
64 bytes from 162.255.119.208: seq=7 ttl=43 time=164.167 ms
64 bytes from 162.255.119.208: seq=8 ttl=43 time=164.081 ms
64 bytes from 162.255.119.208: seq=9 ttl=43 time=164.236 ms
