FROM ubuntu:22.04

USER root

ARG TARGETARCH=amd64

ENV NB_USER=jovyan
ENV NB_UID=1000
ENV NB_GID=0
ENV NB_PREFIX=/
ENV HOME=/home/$NB_USER
ENV SHELL=/bin/bash

# the GID of the 'users' group
ENV USERS_GID=100

ENV HOME_TMP=/tmp_home/$NB_USER
ENV S6_CMD_WAIT_FOR_SERVICES_MAXTIME=300000
ENV S6_BEHAVIOUR_IF_STAGE2_FAILS=2
ARG KUBECTL_VERSION=v1.31.6
ARG S6_VERSION=v3.2.0.2

SHELL ["/bin/bash", "-c"]
RUN export DEBIAN_FRONTEND=noninteractive \
 && apt-get -yq update \
 && apt-get -yq install --no-install-recommends \
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
    nano \
    software-properties-common \
    tzdata \
    unzip \
    vim \
    wget \
    xz-utils \
    zip \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

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

RUN useradd -M -N \
    --shell /bin/bash \
    --home ${HOME} \
    --uid ${NB_UID} \
    --gid ${NB_GID} \
    --groups ${USERS_GID} \
    ${NB_USER} \
 && mkdir -pv ${HOME} \
 && mkdir -pv ${HOME_TMP} \
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

ARG JUPYTERLAB_VERSION=4.3.5
ARG JUPYTER_VERSION=7.3.2
ARG MINIFORGE_VERSION=24.11.3-0
ARG NODE_MAJOR_VERSION=20
ARG PIP_VERSION=24.3.1
ARG PYTHON_VERSION=3.12.13

USER root

# install -- node.js
RUN export DEBIAN_FRONTEND=noninteractive \
 && curl -sL "https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key" | apt-key add - \
 && echo "deb https://deb.nodesource.com/node_${NODE_MAJOR_VERSION}.x nodistro main" > /etc/apt/sources.list.d/nodesource.list \
 && apt-get -yq update \
 && apt-get -yq install --no-install-recommends \
    nodejs \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# setup environment for conda
ENV CONDA_DIR=/opt/conda
ENV PATH="${CONDA_DIR}/bin:${PATH}"
RUN mkdir -pv ${CONDA_DIR} \
 && chmod 2775 ${CONDA_DIR} \
 && echo ". /opt/conda/etc/profile.d/conda.sh" >> ${HOME}/.bashrc \
 && echo ". /opt/conda/etc/profile.d/conda.sh" >> /etc/profile \
 && echo "conda activate base" >> ${HOME}/.bashrc \
 && echo "conda activate base" >> /etc/profile \
 && chown -R ${NB_USER}:${NB_GID} ${CONDA_DIR} \
 && chown -R ${NB_USER}:${USERS_GID} ${HOME}

# create the SYSTEM_CONFIG_PATH for jupyter, and make it owned by NB_USER
# this is needed for jupyter to write `--level=system` configs
RUN mkdir -pv /usr/local/etc/jupyter \
 && chown -R ${NB_USER}:${NB_GID} /usr/local/etc/jupyter

# switch to NB_UID for installs
USER $NB_UID

# install - conda, pip, python, jupyter
RUN case "${TARGETARCH}" in \
      amd64) MINIFORGE_ARCH="x86_64" ;; \
      arm64) MINIFORGE_ARCH="aarch64" ;; \
      ppc64le) MINIFORGE_ARCH="ppc64le" ;; \
      *) echo "Unsupported architecture: ${TARGETARCH}"; exit 1 ;; \
    esac \
 && curl -fsSL "https://github.com/conda-forge/miniforge/releases/download/${MINIFORGE_VERSION}/Miniforge3-${MINIFORGE_VERSION}-Linux-${MINIFORGE_ARCH}.sh" -o /tmp/Miniforge3.sh \
 && curl -fsSL "https://github.com/conda-forge/miniforge/releases/download/${MINIFORGE_VERSION}/Miniforge3-${MINIFORGE_VERSION}-Linux-${MINIFORGE_ARCH}.sh.sha256" -o /tmp/Miniforge3.sh.sha256 \
 && echo "$(cat /tmp/Miniforge3.sh.sha256 | awk '{ print $1; }')  /tmp/Miniforge3.sh" | sha256sum -c - \
 && rm /tmp/Miniforge3.sh.sha256 \
 && /bin/bash /tmp/Miniforge3.sh -b -f -p ${CONDA_DIR} \
 && rm /tmp/Miniforge3.sh \
 && conda config --system --set auto_update_conda false \
 && conda config --system --set show_channel_urls true \
 && echo "python ==${PYTHON_VERSION}" >> ${CONDA_DIR}/conda-meta/pinned \
 && conda install -y -q \
    python=${PYTHON_VERSION} \
    pip=${PIP_VERSION} \
 && conda update -y -q --all \
 && conda clean -a -f -y

# install - jupyter
RUN echo "jupyterlab ==${JUPYTERLAB_VERSION}" >> ${CONDA_DIR}/conda-meta/pinned \
 && echo "notebook ==${JUPYTER_VERSION}" >> ${CONDA_DIR}/conda-meta/pinned \
 && conda install -y -q \
    jupyterlab==${JUPYTERLAB_VERSION} \
    notebook==${JUPYTER_VERSION} \
 && conda clean -a -f -y


# s6 - copy scripts
COPY --chown=${NB_USER}:${NB_GID} --chmod=755 s6/ /etc

# configure - jupyter
# NOTE: we use `--level=system` to write these configs at `/usr/local/etc/jupyter` because it defaults
##      to `/opt/conda/etc/jupyter` and conda cleans that directory after every install
RUN jupyter labextension disable --level=system "@jupyterlab/apputils-extension:announcements" \
 && jupyter labextension lock --level=system "@jupyterlab/apputils-extension"

# s6 - 01-copy-tmp-home
# NOTE: the contents of $HOME_TMP are copied to $HOME at runtime
#       this is a workaround because a PVC will be mounted at $HOME
#       and the contents of $HOME will be hidden
RUN cp -p -r -T "${HOME}" "${HOME_TMP}" \
    # give group same access as user (needed for OpenShift)
 && chmod -R g=u "${HOME_TMP}"

EXPOSE 8888

# -- PyTorch CUDA
ARG PYTORCH_VERSION=2.5.1
ARG TORCHAUDIO_VERSION=2.5.1
ARG TORCHVISION_VERSION=0.20.1

ENV NVIDIA_VISIBLE_DEVICES=all
ENV NVIDIA_DRIVER_CAPABILITIES=compute,utility
ENV NVIDIA_REQUIRE_CUDA="cuda>=12.1"


USER $NB_UID

RUN python3 -m pip install --quiet --no-cache-dir --index-url https://download.pytorch.org/whl/cu121 \
    torch==${PYTORCH_VERSION} \
    torchaudio==${TORCHAUDIO_VERSION} \
    torchvision==${TORCHVISION_VERSION}

RUN mamba install -y -q \
    bokeh==3.6.3 \
    cloudpickle==3.1.1 \
    dill==0.3.9 \
    ipympl==0.9.6 \
    matplotlib==3.10.0 \
    numpy==1.26.4 \
    pandas==2.2.3 \
    scikit-image==0.25.1 \
    scikit-learn==1.6.1 \
    scipy==1.15.1 \
    seaborn==0.13.2 \
    xgboost==2.1.4 \
 && mamba clean -a -f -y

ENTRYPOINT ["/init"]

# install - requirements.txt
COPY --chown=${NB_USER}:${NB_GID} requirements.txt /tmp/requirements.txt
RUN python3 -m pip install -r /tmp/requirements.txt --quiet --no-cache-dir \
 && rm -f /tmp/requirements.txt
