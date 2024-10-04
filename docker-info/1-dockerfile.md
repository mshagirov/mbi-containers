# Building, Tagging, and Publishing Docker Images

## 1. Writing a Dockerfile

> Also, see Docker init [documentation](https://docs.docker.com/reference/cli/docker/init/) for initialising a new Docker projects.

### 1.1. Base Image

All Dockerfiles start with base images. Your image then extends this base image.

```Dockerfile
FROM ubuntu:18.04
```

**Good Practice**:

To ensure the base image is secure and from a trusted source, look for "Official Images", "Verified Publisher", and "Docker-Sponsored Open Source" labels on Docker Hub (or similar labels on other registries).

### 1.2. `scratch` Image

Typically used for creating minimal images containing only the application (i.e. an executable app w/o runtime dependencies).

```Dockerfile
FROM scratch
```

For usage example, see "multi-stage builds" below or in the Docker documentation ([link](https://docs.docker.com/build/building/multi-stage/#use-multi-stage-builds)).

Full (base) images can be created using `scratch` and linux distribution packages as a `tar`. E.g., `debootstrap` can be used to package Debian and Ubuntu filesystems for building Docker images.

```bash
sudo debootstrap focal focal > /dev/null
sudo tar -C focal -c . | docker import - focal
```

### 1.3. Build Cache

When executing instruction in Dockerfile, Docker checks if it can reuse instructions from a previous build (cache). If the instruction was already executed before, Docker doesn't execute it again and instead will use the cached result (layers). This is useful for speeding up the build process.

**Cache Invalidation** (situations that cause cache invalidation):

- `RUN` instruction layers. *Any changes to the command* invalidate the cache.
- `COPY` and `ADD` instructions invalidate the cache if *the source file changes*.
  - To optimise the build process, copy only the necessary files and directories and separate the files that change frequently from the files that change rarely. This way, you can reuse the cache for the files that don't change frequently and save building time.
- If the layer is invalidated, *Docker will rebuild the layer and all subsequent layers*.

### 1.4. Multi-Stage Builds

To optimise the final image size and build time, you can use multi-stage builds. This involves using multiple `FROM` statements in a single Dockerfile and copying files between them.

```Dockerfile
FROM golang:1.7.3
WORKDIR /src
COPY . . # copy /src/main.go
RUN go build -o /bin/hello ./main.go

FROM scratch
COPY --from=0 /bin/hello /bin/hello
CMD ["/bin/hello"]
```

Here, `--from=0` refers to the first build Dockerfile. The line,

```Dockerfile
COPY --from=0 /bin/hello /bin/hello
```

only copies the the built artifact `/bin/hello` from the previous stage (integer numbered, here `0`) into this new stage.

The builds also can be named with `AS` statement(s) for better readability.

```Dockerfile
FROM golang:1.7.3 AS build
...
```

Then, refer to the named build with `--from=build`, where `build` is the name of the build.

```Dockerfile
FROM scratch
COPY --from=build /bin/hello /bin/hello
CMD ["/bin/hello"]
```

You can also stop the build when the final build is not needed (e.g., testing or debugging stages) at a certain stage by using `--target` flag with `docker build` command.

```bash
docker build --target=build -t myapp .
```

Copying artifacts from builds is not limited to the Dockerfile. You can also copy build artifacts from external images (local or on a registry).

```Dockerfile
COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf
```

### 1.5. Other Dockerfile and Image Notes and Good Practices

- Don't install unnecessary packages. This reduces the complexity and size of the image and speeds up the build process.
- Separate the applications into different images; one process per image. This allows you to reuse containers and scale them horizontally.
- Sort multi-line arguments alphanumerically. This makes it easier to read and maintain the Dockerfile. Also, use space before the backslash (`\`) for better readability.
- Update apt-get and install packages in the same `RUN` command to avoid caching issues; if the update is run as a separate command, it will be cached by the builder and not run again. Additionally, remove the apt cache after installing packages to reduce the image size (`rm -rf /var/lib/apt/lists/*`). Some official images automatically run `apt-clean` so explicit invocation is not necessary (this should be reflected in the image file size, w/ and w/o explicit chache removal should result in the same image size).

```Dockerfile
RUN apt-get update && apt-get install -y \
    aufs-tools \
    automake \
    build-essential \
    ...
    reprepro \
    ruby1.9.1 \
    ruby1.9.1-dev \
    s3cmd=1.1.* \
    && rm -rf /var/lib/apt/lists/*
```

- `RUN` commands with pipes (`|`) are executed with `/bin/sh -c` interpreter (like other `RUN` commands). This interpreter *only* evaluates the exit code of the last operation in the pipe. In order check and fail the pipe command on any error in the pipe, use `set -o pipefail &&` before the pipe command.

```Dockerfile
RUN set -o pipefail && wget -O - https://some.site | wc -l > /number
```

On debian-based images, the dash shell doesn't support `pipefail` option. To use `pipefail` option, use `bash` shell using the *exec* form `RUN`.

```Dockerfile
RUN ["/bin/bash", "-c", "set -o pipefail && wget -O - https://some.site | wc -l > /number"]
```

## 2. Ephimeral Containers

Images defined by Dockerfiles should be as close as possible to ephemeral containers. *The ephemeral container can be stopped and destroyed, then rebuilt and replaced with an absolute minimum set up and configuration*.

## 3. Keeping Up-to-Date and Rebuilding Images Frequently

Docker image is a snapshot of the image and base images, the libraries, and other software at the time of building. To keep images secure and up-to-date, we need to rebuild images often. To ensure that we are using the latest version of the base image with the latest security patches and library versions, we can use the `--no-cache` flag with the `docker build` command.

Although images are immutable, the tags are not and they can be modified; e.g., tag `ubuntu:24.04` might evolve to point to different versions of `ubuntu`, s.a. later version `ubuntu:24.04.3`.

```bash
docker build --no-cache -t IMAGE_NAME:TAG .
```

> "Cache hits": failing to use the latest version of the base image due to the stored cache of an older version of the base image(s).

For some cases, we might want to ensure that we are using an exact version of an image (e.g., for a completed project). To guarantee that you get an exact copy of the image, use *pinning* with the SHA256 hash of the image. This is especially useful when you want to ensure that you want to control the exact version/variant of an image and you are using the same image in different environments (e.g., development, testing, and production).

```bash
# syntax=docker/dockerfile:1
FROM alpine:3.19@sha256:13b7e62e8df80264dbb747995705a986aa530415763a6c58f84a3ca8af9a5bcd
```

With this Dockerfile, your build will always use the *pinning* image version `sha256:13b7e62e8...` even if the publisher updates the `3.19` tag of the base image. *Pinning base image(s) trades automatic updates for control over the image version*.
