# syntax=docker/dockerfile:1

FROM ghcr.io/kubeflow/kubeflow/notebook-servers/codeserver:sha-4a26c7b5e9575410613faf7df6735aa1883a2d24 AS base
USER root

# Install system dependencies
ENV TZ="Asia/Singapore"
RUN apt-get update && DEBIAN_FRONTEND="noninteractive" apt-get -y install tzdata
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    apt-utils \
    apt-transport-https \
    ca-certificates \
    gnupg \
    nano \
    sudo \
    wget

# Add signing key to install GitHub CLI: https://github.com/cli/cli/blob/trunk/docs/install_linux.md
RUN wget -O /etc/apt/keyrings/github-cli.gpg https://cli.github.com/packages/githubcli-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/github-cli.gpg] https://cli.github.com/packages stable main" >> /etc/apt/sources.list.d/github-cli.list

ARG BASE=jammy

# Install the Chaste repo list and key: https://chaste.github.io/docs/installguides/ubuntu-package/
RUN wget -O /usr/share/keyrings/chaste.asc https://chaste.github.io/chaste.asc \
    && echo "deb [signed-by=/usr/share/keyrings/chaste.asc] https://chaste.github.io/ubuntu ${BASE}/" >> /etc/apt/sources.list.d/chaste.list

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    chaste-dependencies \
    clang \
    lldb \
    gdb \
    gh \
    valgrind \
    "libpetsc-real*-dbg" \
    hdf5-tools \
    cmake-curses-gui \
    doxygen \
    graphviz && \
    rm -rf /var/lib/apt/lists/*

# Update system to use Python3 by default
RUN update-alternatives --install /usr/bin/python python /usr/bin/python3 1

# install for USER uid=1000, gid=100
# Set environment variables with args to allow for changes at build time
ARG USER="jovyan"
ARG UID=1000
ARG GID=100
ARG CHASTE_DIR="/home/${USER}"
ARG CMAKE_BUILD_TYPE="Debug"
ARG Chaste_ERROR_ON_WARNING="ON"
ARG Chaste_UPDATE_PROVENANCE="OFF"
ENV USER=${USER} \
    GROUP=${USER} \
    UID=${UID} \
    GID=${GID} \
    CHASTE_DIR=${CHASTE_DIR} \
    CMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} \
    Chaste_ERROR_ON_WARNING=${Chaste_ERROR_ON_WARNING} \
    Chaste_UPDATE_PROVENANCE=${Chaste_UPDATE_PROVENANCE} \
    CHASTE_SOURCE_DIR="${CHASTE_DIR}/src" \
    CHASTE_BUILD_DIR="${CHASTE_DIR}/build" \
    CHASTE_TEST_OUTPUT="${CHASTE_DIR}/output" \
    PATH="${CHASTE_DIR}/scripts:${PATH}"
ENV CHASTE_PROJECTS_DIR="${CHASTE_SOURCE_DIR}/projects" \
    TEXTTEST_HOME="${CHASTE_BUILD_DIR}/texttest_venv" \
    PYTHONPATH="${CHASTE_BUILD_DIR}/python"

# RUN userdel -r ubuntu || true
COPY --chown=${USER}:${GID} scripts "${CHASTE_DIR}/scripts"

# set user to jovyan (uid=1000)
USER 1000
WORKDIR ${CHASTE_DIR}

# Chaste(C): TextTest for regression testing (requires pygtk)
RUN python -m venv --upgrade-deps "${CHASTE_BUILD_DIR}/texttest_venv" && \
    . "${CHASTE_BUILD_DIR}/texttest_venv/bin/activate" && \
    python -m pip install --no-cache-dir texttest

RUN mkdir -p "${CHASTE_SOURCE_DIR}" "${CHASTE_BUILD_DIR}" "${CHASTE_TEST_OUTPUT}" && \
    ln -s "${CHASTE_PROJECTS_DIR}" projects

# C: Fix git permissions issue CVE-2022-24765
RUN git config --global --add safe.directory "${CHASTE_SOURCE_DIR}"
# C: Save Chaste version and dependencies information
RUN apt-cache show chaste-dependencies > "${CHASTE_TEST_OUTPUT}/chaste-dependencies.txt"
# RUN git clone --depth 1 --branch release_2021.1 https://github.com/Chaste/Chaste.git

# To install PyChaste you may use mamba:
# NOTE: mamba python is not compatible with the system python
#       !!! build a separate image for PyChaste, e.g. from codeserver-python (includes mamba/conda)!!!
# RUN mamba install -y -q \
#     -c pychaste -c conda-forge chaste \
#     && mamba clean -a -f -y

# NOTE: Kubeflow
# s6 - 01-copy-tmp-home
# the contents of $HOME_TMP are copied to $HOME at runtime
# this is a workaround because a PVC will be mounted at $HOME
# and the contents of $HOME will be hidden
RUN cp -p -r -T "${HOME}" "${HOME_TMP}" \
    # give group same access as user (needed for OpenShift)
 && chmod -R g=u "${HOME_TMP}"

# --------------------------------------------------------------------------------
FROM base AS build

USER ${USER}
# C: Build Chaste: GIT_TAG can be a branch or release ('-' skips by default)
ARG GIT_TAG=release_2021.1 
ENV GIT_TAG=${GIT_TAG}
RUN build_chaste.sh ${GIT_TAG}

# C: Automatically mount the home directory in a volume to persist changes made there.
#  NOTE: After declaring the volume, changes to the contents during build will not persist.
# VOLUME "${CHASTE_DIR}"
# NOTE: Kubeflow
# s6 - 01-copy-tmp-home
# the contents of $HOME_TMP are copied to $HOME at runtime
# this is a workaround because a PVC will be mounted at $HOME
# and the contents of $HOME will be hidden
RUN cp -p -r -T "${HOME}" "${HOME_TMP}" \
    # give group same access as user (needed for OpenShift)
 && chmod -R g=u "${HOME_TMP}"

# --------------------------------------------------------------------------------
# FROM build AS test
# USER ${USER}
# ARG TEST_SUITE=-
# ENV TEST_SUITE=${TEST_SUITE}
# RUN test.sh ${TEST_SUITE}
#
