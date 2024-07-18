# 1. Getting Started

## 1.1. Docker Basics

**Container**

- Container runs in a custom isolated filesystem (fs)
  - *container fs* is provided by the container image (see below)
  - *container fs* must contain everything needed to run an application; all the dependencies, config-s, scripts, and binaries (executables) etc.
- Due to the isolated fs, containers are portable and consistent across environments/machines that support containers.
- Isolation allows running multiple containers with different dependencies on the same host machine.

**Container Image (a.k.a. container's "blueprint"):**

- Container gets all its files and configuration from the image a.k.a. the *container image*.

- A container image also contains environment variables and a default command to run when container starts.

- Images are *immutable* and cannot be changed after creation. You can make changes by creating a new image or using the image as a base for a new image (layer images, see next).

- Container images are composed of *layers** (of filesystems), where each layer modifies the previous fs layer and adds or removes files.
  
  - You can list the layers of an image with `docker history <image-name>`, e.g. 
    
    ```bash
    docker history ubuntu:18.04
    ```

**Container Image Registry**

- Although the words registry and repository are often used interchangeably, they have different meanings. A *registry* is a centralised location to *store and manage container images*, whereas a *repository is a collection of related container images* within the registry.
  - E.g.,

```yaml
registry:
 repository1:
    - name: "Project A"
    - image-v1.0
    - image-v1.1
 repository2:
    - name: "Project B"
    - image-v1.0
    - image-v1.1
    - image-v2.0
```

**Dockerfile**

> A Text file that contains a set of all the commands required for creating a container *image*.<br/>
> Dockerfile reference : https://docs.docker.com/reference/dockerfile/ <br/>
> Also see: `docker init` documentation

- Common instructions in a Dockerfile:
  
  - `FROM <image>` : specifies the base image that the built image will extend.
  
  - `WORKDIR <path>` : sets the working directory, the path in the image where the file copied to and commands executed.
  
  - `COPY <src> <dest>` : copies files from the host to the image. 
  
  - `RUN <command>` : runs a command in the image (in working directory).
  
  - `ENV <name> <value>` : sets environment variables in the image.
  
  - `EXPOSE <port>` : sets configuration on the image that indicates the port image will expose (when running).
  
  - `USER <username>` : sets the default user for subsequent instructions; might rerquire `RUN useradd <username>` before setting the default user.
  
  - `CMD <command> <arg1> <arg2> ...` : sets the default command to run when the container starts.
  
  - `ENTRYPOINT <executable> <arg1> <arg2> ...` : sets the default executable to run when the container starts. Use this when you want to run container as an executable.
    
    - *Dockerfile should specify at least one of `ENTRYPOINT` or `CMD` commands*.
    
    - If the `CMD` is defined in the base image, `ENTRYPOINT` will reset `CMD` to an empty value.
    
    - `ENTRYPOINT` executable runs as `/bin/sh -c` and reset the shell environment variables and ignore `CMD` and `docker run` arg-s (for shell processing shown above). To ensure that `docker stop` will signal any long running `ENTRYPOINT` executable correctly, you need to remember to start it with `exec` (the command will get PID 1 which allows it to receive stop signals correctly):
      
      ```Dockerfile
      ENTRYPOINT exec <command> <arg-s>
      ```

**Exceptions**

- Although containers are not affected by the host's environment, they can still fail or behave unexpectedly due to the updates to docker engine, or the host's kernel, or the host's environment.

# 2. Containarising an App with Docker

> If the app consists of multiple services, it's better to use multiple containers with each service in a separate container. In such cases, use docker-compose or k8s to manage (a.k.a. orchestarte) the services.<br />
> *We desire each container to do one thing and do it well.*

## 2.1. Build a container image for the App

**Example Dockerfile** 

E.g. container for a NodeJS app, create a text file (the Dockerfile) in the app directory that contains package configs and dependencies (`package.json`)

```bash
touch Dockerfile # create an empty file
```

with the following contents (instructions for building a container image)

```bash
# syntax=docker/dockerfile:1

FROM node:18-alpine  # starting image
WORKDIR /app         # root dir-y
COPY . .             # copy "app" dir to the (image) root dir
RUN yarn install --production # install depen-s
CMD ["node", "src/index.js"] # default command to run command+options
EXPOSE 3000
```

then build the container image with:

```bash
cd /path/to/app # optional navigate to the location of the docker file

docker build -t getting-started .
```

the `-t` tags the image with human-readable name (here, "getting-started"), ` . ` is the location of the `Dockerfile`.

## 2.2. Starting the app container

Start the container with the  `docker run`:

```bash
docker run -dp 3000:3000 getting-started
```

the `-d` runs the container in detached mode (background), `-p` flag maps the host's port 3000 to the container's port 3000, and allows access to the application for the host.

## 2.3. List running containers

```bash
docker ps
```

the output should be something like this

```
# output:
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
ed11cdbe146c        getting-started     "docker-entrypoint.s…"   24 minutes ago      Up 24 minutes       0.0.0.0:3000->3000/tcp   cocky_diffie
```

## 2.4. Updating the source code of the app

To modify the source code and update an image of the app:

1. edit the source code and modify.
2. re-build the image with `docker build`
3. start the new container (,if the port available. This requires stopping the old container).

### 2.4.1. Removing the old container

1. List the containers and determine its ID:
   
   ```bash
   docker ps
   ```

2. Stop the container
   
   ```bash
   docker stop <the-container-id>
   ```

3. Once the container has stopped, you can remove it
   
   ```bash
   docker rm <the-container-id>
   ```

or stop and remove the container in a single command with a force flag `-f`

```bash
docker rm -f <the-container-id>
```

# 3. Sharing the image

Create a repository on Docker Hub. Name your repository (e.g., "getting-started").

## 3.1. Push the image

1. Login 
   
   ```bash
   docker login -u YOUR-USER-NAME
   ```

2. Give the local image "getting-started" a new name with `docker tag` ([documentation](https://docs.docker.com/engine/reference/commandline/tag/))
   
   ```bash
   docker tag getting-started YOUR-USER-NAME/getting-started
   ```

3. Now try your push command. If you’re copying the value from Docker Hub, you can drop the `tagname` portion, as you didn’t add a tag to the image name. If you don’t specify a tag, Docker will use a tag called `latest`.
   
   ```
   docker push YOUR-USER-NAME/getting-started
   ```

4. You can see the image tag with:
   
   ```bash
   docker image ls
   ```
   
   Default tag when you don't specify is "latest".

# 4. Running Interactive Containers

```
docker run --name CONTAINER_NAME -it IMAGE_NAME # "--name CONTAINER_NAME" is optional
```

runs container image `IMAGE_NAME`, and `-t` flag allocates a pseudo-TTY, `-i` sets interactive mode, `--name` gives the container a name.

# 5. Automating Container Builds and Deployment

## 5.1. `ARG` and `ENV` in Dockerfile

**Quick reference:**

- `ARG` is used to pass arguments to the Dockerfile *at build time*. E.g. you can use it to pass arguments to the `RUN` command. `ARG`s are build-time arguments.

- `ENV` is used to set environment variables in the container *at runtime*. This can be used for setting environment variables for the container, e.g., for `CMD` and `ENTRYPOINT` commands.

- `ENV` is also available for the container during build time starting from line you define it.

- `ARG` values can be used to set `ENV` values.
  
  ```Dockerfile
  ARG buildtime_variable=default_value # define ARG variable for build time
  ENV env_var_name=$buildtime_variable # reference directly in ENV
  ```
  
  then override the `ARG` value at build time:
  
  ```bash
  docker build --build-arg buildtime_variable=a_value 
  ```
  
  (this allows changing the value of `env_var_name` at build time w/o editing the Dockerfile)

- The default value of and `ARG` variable is optional, but if not provided, the build will fail if the `ARG` is not provided at build time.

## 5.2. Docker-compose, Volumes, and Multi-Container Apps

> Volumes are used for persistent data storage or sharing data between the host and the container and between containers.<br/>*See example app here* https://github.com/dockersamples/todo-list-app

- Volumes can be mounted as a command line argument (for short commands or for testing)

- For applications use `docker-compose`; volumes are defined and managed in the `docker-compose.yml` file (`*.yml`).

- In the directory where the `docker-compose.yml` file is located, run the following command to start the app:
  
  ```bash
  docker-compose up
  ```
  
  you can re-run this command if you make changes to the `docker-compose.yml` file, all changes will be applied automatically. Docker-compose will create the volumes and networks automatically if they don't exist (based on the `.yml` file).

- To stop the app, run:
  
  ```bash
   docker-compose down
  ```

- To remove the volumes, run:
  
  ```bash
  docker-compose down --volumes
  ```
  
  Volumes are not removed by default when you run `docker-compose down` because they contain data that you might want to keep and reuse, e.g. when you are developing the containers or apps.

## 5.3. `.env` file Docker-compose

> e.g., when you have a more complex app with multiple services, you can use docker-compose to define and manage multiple containers with different services.

- `.env` file uses key-value pairs to define environment variables. Docker-compose uses `.env` file to set environment variables that you define in the `docker-compose.yml` file.

- E.g.: in a `.env` file, we define:
  
  ```Dockerfile
  VARIABLE_NAME=some variable value
  ```

- Then, we can set and use an environment variable in the `docker-compose.yml` file:
  
  ```yaml
  version: '3'  
  services:
     example:
     image: ubuntu
     environment:
        - env_var_name=${VARIABLE_NAME} #<--set env_var_name=VARIABLE_NAME
  ```


