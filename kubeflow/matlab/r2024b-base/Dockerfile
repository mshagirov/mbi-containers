# Modified Dockerfile from Copyright 2023-2024 The MathWorks, Inc.
# Murat Shagirov, 2024

# Kubeflow requirements:
#   - container image must expose HTTP interface on port 8888 and set NB_PREFIX as a proxy for Matlab apps 
#   - run as "jovyan (uid=1000)" (i.e., need to rename user "matlab" to jovyan)
#   - jovyan's home: "/home/jovyan", mounted at start as PVC.
#   - Since some of matlab apps & config in home folder /home/matlab (no need to change), mount jovyan separately for PVC storage.


# To specify which MATLAB release to install in the container, e.g.: ARG MATLAB_RELEASE=R2021b
ARG MATLAB_RELEASE=R2024b

# Ubuntu-based mathworks/matlab image.
# For more matlab images, see: https://hub.docker.com/r/mathworks/matlab
FROM mathworks/matlab:$MATLAB_RELEASE

# declare MATLAB_RELEASE to reuse the arg from before FROM
ARG MATLAB_RELEASE

# By default, the MATLAB container runs as user "matlab". To install mpm dependencies, switch to root.
USER root
ARG TARGETARCH=amd64

# Kubeflow : common env vars
# ENV NB_USER=jovyan
ENV NB_USER=matlab
ENV NB_UID=1000
# WARNING: the primary GID of 'jovyan' MUST be 0!
#          this allows any UID to be used as all important folders are owned by GID 0.
#          some Kubernetes environments run containers as a random UID (e.g. OpenShift).
#          note, having GID 0 (root) does NOT give you root permissions, so this is not a security issue.
ENV NB_GID=0
ENV NB_PREFIX=/
ENV HOME=/home/$NB_USER
ENV SHELL=/bin/bash
# the GID of the 'users' group
ENV USERS_GID=100

# The contents of $HOME_TMP copied to $HOME on startup
# to mount PVC to $HOME
ENV HOME_TMP=/tmp_home/$NB_USER

# s6-overlay only gives 5 seconds by default, which is too small for slow PVC storage backends
# when running `/etc/cont-inid.d/01-copy-tmp-home` (note, this is in milliseconds)
ENV S6_CMD_WAIT_FOR_SERVICES_MAXTIME=300000
# s6-overlay does not fail by default if the `/etc/cont-init.d/` or `/etc/services.d/` scripts fail
# this is not the desired behavior, so we set it to fail
ENV S6_BEHAVIOUR_IF_STAGE2_FAILS=2

ARG KUBECTL_VERSION=v1.29.9
ARG S6_VERSION=v3.2.0.0

# set shell to bash
SHELL ["/bin/bash", "-c"]

# install - usefull linux packages
RUN export DEBIAN_FRONTEND=noninteractive \
 && apt-get clean \
 && apt-get update \
 && apt-get install --no-install-recommends --yes\
    apt-transport-https \
    bash \
    bzip2 \
    ca-certificates \
    curl \
    git \
    gnupg \
    gnupg2 \
    locales \
    lsb-release \
    python3 \
    python3-pip \
    software-properties-common \
    tzdata \
    unzip \
    wget \
    xvfb \
    xz-utils \
    zip \
 && apt-get clean \
 && apt-get autoremove \
 && rm -rf /var/lib/apt/lists/*

# install - s6 overlay
RUN case "${TARGETARCH}" in \
      amd64) S6_ARCH="x86_64" ;; \
      arm64) S6_ARCH="aarch64" ;; \
      ppc64le) S6_ARCH="ppc64le" ;; \
      *) echo "Unsupported architecture: ${TARGETARCH}"; exit 1 ;; \
    esac \
 && curl -fsSL "https://github.com/just-containers/s6-overlay/releases/download/${S6_VERSION}/s6-overlay-noarch.tar.xz" -o /tmp/s6-overlay-noarch.tar.xz \
 && curl -fsSL "https://github.com/just-containers/s6-overlay/releases/download/${S6_VERSION}/s6-overlay-noarch.tar.xz.sha256" -o /tmp/s6-overlay-noarch.tar.xz.sha256 \
 && echo "$(cat /tmp/s6-overlay-noarch.tar.xz.sha256 | awk '{ print $1; }')  /tmp/s6-overlay-noarch.tar.xz" | sha256sum -c - \
 && curl -fsSL "https://github.com/just-containers/s6-overlay/releases/download/${S6_VERSION}/s6-overlay-${S6_ARCH}.tar.xz" -o /tmp/s6-overlay-${S6_ARCH}.tar.xz \
 && curl -fsSL "https://github.com/just-containers/s6-overlay/releases/download/${S6_VERSION}/s6-overlay-${S6_ARCH}.tar.xz.sha256" -o /tmp/s6-overlay-${S6_ARCH}.tar.xz.sha256 \
 && echo "$(cat /tmp/s6-overlay-${S6_ARCH}.tar.xz.sha256 | awk '{ print $1; }')  /tmp/s6-overlay-${S6_ARCH}.tar.xz" | sha256sum -c - \
 && tar -C / -Jxpf /tmp/s6-overlay-noarch.tar.xz \
 && tar -C / -Jxpf /tmp/s6-overlay-${S6_ARCH}.tar.xz \
 && rm /tmp/s6-overlay-noarch.tar.xz  \
       /tmp/s6-overlay-noarch.tar.xz.sha256 \
       /tmp/s6-overlay-${S6_ARCH}.tar.xz \
       /tmp/s6-overlay-${S6_ARCH}.tar.xz.sha256

RUN chmod 0775 /run

# install - kubectl
RUN curl -fsSL "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${TARGETARCH}/kubectl" -o /usr/local/bin/kubectl \
 && curl -fsSL "https://dl.k8s.io/${KUBECTL_VERSION}/bin/linux/${TARGETARCH}/kubectl.sha256" -o /tmp/kubectl.sha256 \
 && echo "$(cat /tmp/kubectl.sha256 | awk '{ print $1; }')  /usr/local/bin/kubectl" | sha256sum -c - \
 && rm /tmp/kubectl.sha256 \
 && chmod +x /usr/local/bin/kubectl

# create user and set required ownership
RUN usermod \
    -s /bin/bash \
    --gid ${NB_GID} \
    --groups ${USERS_GID} \
    --groups ${NB_UID} \ 
    matlab \
 && mkdir -pv ${HOME} \
 && mkdir -pv ${HOME_TMP} \
    # in the interest of backwards compatibility we have the 'users' group owns the home directory
    # we also set the SGID bit so that new files and directories are created with the 'users' group
 && chmod 2775 ${HOME} \
 && chmod 2775 ${HOME_TMP} \
 && chown -R ${NB_USER}:${USERS_GID} ${HOME} \
 && chown -R ${NB_USER}:${USERS_GID} ${HOME_TMP} \
 && chown -R ${NB_USER}:${NB_GID} /usr/local/bin

# set locale configs
RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen \
 && locale-gen
ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

# s6 - copy scripts
COPY --chown=${NB_USER}:${NB_GID} --chmod=755 s6/ /etc

EXPOSE 8888

ENV NB_PREFIX=/
ENV MWI_APP_PORT=8888
ENV MWI_BASE_URL=${NB_PREFIX}

COPY --chown=0:0 --chmod=755 ./start_matlab /usr/bin/

# USER matlab
USER $NB_UID

ENTRYPOINT ["tini", "--","start_matlab"]

WORKDIR ${HOME}