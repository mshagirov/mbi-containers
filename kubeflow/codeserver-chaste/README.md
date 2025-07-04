# Dockerfile for VS Code Server with Chaste

- Built from [codeserver:sha-4a26c7b](https://ghcr.io/kubeflow/kubeflow/notebook-servers/codeserver:sha-4a26c7b5e9575410613faf7df6735aa1883a2d24)
- References default Chaste image (latest release) [chaste/release](https://github.com/Chaste/chaste-docker/blob/master/Dockerfile)

For container image repo "muratshagirov" and image name "codeserver-chaste"
(following Kubeflow's naming conventions), build the image with:

```bash
TAG=develop # git tag for the release
REPO=muratshagirov # repo name, e.g. on Dockerhub
IMAGE=codeserver-chaste # image's name
TARGET="linux/amd64" # target system (current Kubeflow system setup)
docker build --platform $TARGET --build-arg GIT_TAG=$TAG -t $REPO/$IMAGE:$TAG .
```

Tested git tags (Chaste releases):

- `develop`
- `2024.2`

> Release version/tag depends on the OS version, e.g., above tags work for Ubuntu
22.04 LTS (Jammy) but older Chaste releases might not compile due to mismatching
system package names and the Chaste building script.
