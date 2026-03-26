# CAREamics

> Dockerfiles for Kubeflow notebook images (v1)

- For building locally,

```bash
### Build (and push) the base image with py3.12 and CUDA 12.1
docker build --platform linux/amd64 -t muratshagirov/jupyter-pytorch-cu121 ./base
### E.g.,
# docker push muratshagirov/jupyter-pytorch-cu121
#
### Build an image with CAREamics
docker build --platform linux/amd64 -t muratshagirov/careamics-cu121-py312 .
```

- For using online repo, push the base image to an online repository. Then, edit
the CAREamics Dockerfile to use the base image repository with SHA256 digest,
e.g.,

```Dockerfile
FROM docker.io/muratshagirov/jupyter-pytorch-cu121:latest@sha256:5cfefe9463e7b2b3271af42f2c6a26acf4bc532df8775fac1bd85ac0c30157b2
# install CAREamics ...
```
