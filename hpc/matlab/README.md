# Dockerfile for Matlab container with custom addons

- For more examples and containers read Matlab git repo README [here](https://github.com/mathworks-ref-arch/matlab-dockerfile/tree/main/alternates/building-on-matlab-docker-image).

- You may also like [Getting Started with Matlab in Docker](https://github.com/mathworks-ref-arch/getting-started-with-matlab-in-docker).

- For Matlab containers with GPU support (Deep Learning), read [here](https://hub.docker.com/r/mathworks/matlab-deep-learning).

## Build the image

- Build with default Matlab version and addons in the Dockerfile

```bash
# git clone mbi-containers
# cd mbi-containers/hpc/matlab
docker build -t matlab_with_add_ons:R2024b . # assuming the default release is R2024b
```

- Build with custom Matlab version and addons:

```bash
# git clone mbi-containers
# cd mbi-containers/hpc/matlab
docker build --build-arg MATLAB_RELEASE=R2022b -t matlab_with_add_ons:R2022b .
```

Note that "ADDITIONAL_PRODUCTS" build-time argument has been removed (commented out) from the Dockerfile. Instead modify the line that writes `/tmp/products` to include the additional products you want to install. This makes it easier to keep track of the installed products after the image is built.

## Run the container

- Run the container with a browser-based interface:

```bash
docker run --init --rm -it -p 8888:8888 matlab_with_add_ons:R2024b -browser
```

- Run the container with a CLI interface:

```bash
docker run -it --rm --shm-size=512M matlab_with_add_ons:R2024b
```

- Run the container with a VNC interface (virtual desktop environment):

```bash
docker run -it --rm -p 5901:5901 -p 6080:6080 matlab_with_add_ons:R2024b -vnc
```

If you are using VNC client on your machine, enter `localhost:1` when prompted for a VNC password. This password is customisable at the launch.
