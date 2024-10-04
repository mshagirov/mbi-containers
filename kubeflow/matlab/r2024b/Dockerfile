# Modified Dockerfile from Copyright 2023-2024 The MathWorks, Inc.
# Murat Shagirov, 2024

# Kubeflow requirements:
#   - container image must expose HTTP interface on port 8888
#   - run as "jovyan (uid=1000)"
#   - jovyan's home: "/home/jovyan", mounted at start as PVC.


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
    software-properties-common \
    tzdata \
    unzip \
    wget \
    xz-utils \
    zip \
 && apt-get clean \
 && apt-get autoremove \
 && rm -rf /var/lib/apt/lists/*

# RUN echo "arch:${TARGETARCH}"
 
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
#   -l ${NB_USER} \
    -s /bin/bash \
    # -m --home ${HOME} \
    --gid ${NB_GID} \
    --groups ${USERS_GID} \
    --groups ${NB_UID} \ 
    matlab \
#  && echo "${NB_USER}:${NB_GID}" | chpasswd \
#  && echo "${NB_USER} ALL=(ALL) NOPASSWD: ALL"> /etc/sudoers.d/${NB_USER} \
#  && groupmod --new-name ${NB_USER} matlab\
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

# Run mpm to install MathWorks products into the existing MATLAB installation directory,
# and delete the mpm installation afterwards.
# Modify it by setting the ADDITIONAL_PRODUCTS defined above,
# e.g. ADDITIONAL_PRODUCTS="Statistics_and_Machine_Learning_Toolbox Parallel_Computing_Toolbox MATLAB_Coder".
# If mpm fails to install successfully then output the logfile to the terminal, otherwise cleanup.

# Switch to user matlab, and pass in $HOME variable to mpm,
# so that mpm can set the correct root folder for the support packages.
WORKDIR /tmp

# The quotations "" around the package name and the new line \n are required
RUN echo \
"Curve_Fitting_Toolbox "\
"Image_Processing_Toolbox "\
"Deep_Learning_Toolbox "\
"Optimization_Toolbox "\
"Parallel_Computing_Toolbox "\
"Statistics_and_Machine_Learning_Toolbox "\
"Symbolic_Math_Toolbox "\
>> /tmp/products

RUN wget -q https://www.mathworks.com/mpm/glnxa64/mpm \
    && chmod +x mpm \
    && EXISTING_MATLAB_LOCATION=$(dirname $(dirname $(readlink -f $(which matlab)))) \
    && sudo HOME=${HOME} ./mpm install \
        --destination=${EXISTING_MATLAB_LOCATION} \
        --release=${MATLAB_RELEASE} \
        --products $(cat ./products) \
    || (echo "MPM Installation Failure. See below for more information:" && cat /tmp/mathworks_root.log && false) \
    && sudo rm -rf mpm /tmp/mathworks_root.log /tmp/products

# When running the container a license file can be mounted,
# or a license server can be provided as an environment variable.
# For more information, see https://hub.docker.com/r/mathworks/matlab

# Alternatively, you can provide a license server to use
# with the docker image while building the image.
# Specify the host and port of the machine that serves the network licenses
# if you want to bind in the license info as an environment variable.
# You can also build with something like --build-arg LICENSE_SERVER=27000@MyServerName,
# in which case you should uncomment the following two lines.
# If these lines are uncommented, $LICENSE_SERVER must be a valid license
# server or browser mode will not start successfully.
# ARG LICENSE_SERVER
# ENV MLM_LICENSE_FILE=$LICENSE_SERVER

# The following environment variables allow MathWorks to understand how this MathWorks
# product is being used. This information helps us make MATLAB even better.
# Your content, and information about the content within your files, is not shared with MathWorks.
# To opt out of this service, delete the environment variables defined in the following line.
# See the Help Make MATLAB Even Better section in the accompanying README to learn more:
# https://github.com/mathworks-ref-arch/matlab-dockerfile#help-make-matlab-even-better

# ENV MW_DDUX_FORCE_ENABLE=true MW_CONTEXT_TAGS=$MW_CONTEXT_TAGS,MATLAB:TOOLBOXES:DOCKERFILE:V1

EXPOSE 8888

# USER matlab
USER $NB_UID
# for Matlab apps in user's home dir-y
ENV PATH="${HOME}/.local/bin:$PATH"
ENTRYPOINT ["tini", "--", "/bin/run.sh"]
CMD ["-browser"]

WORKDIR ${HOME}

# ENTRYPOINT ["tini" "--" "/bin/run.sh"]