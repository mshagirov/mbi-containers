# Use the official GROMACS image as the base image
FROM gromacs/gromacs:2021.4

ENV MY_USER=jovyan
ENV MY_UID=1000
ENV MY_GID=1000
ENV USERS_GID=100
ENV MY_HOME=/home/${MY_USER}
ENV PATH=/gromacs/bin:$PATH

# Set the working directory
#WORKDIR /usr/local/gromacs

SHELL ["/bin/bash", "-c"]
USER root
RUN groupadd -g ${MY_GID} ${MY_USER} \
 && useradd -M -N \
    --shell /bin/bash \
    --home ${MY_HOME} \
    --uid ${MY_UID} \
    --gid ${MY_GID} \
    --groups ${USERS_GID} \
    ${MY_USER} \
 && mkdir -pv ${MY_HOME} \
    # in the interest of backwards compatibility we have the 'users' group owns the home directory
    # we also set the SGID bit so that new files and directories are created with the 'users' group
 && chmod 2775 ${MY_HOME} \
 && chown -R ${MY_USER}:${USERS_GID} ${MY_HOME} \
 && chown -R ${MY_USER}:${MY_GID} /usr/local/bin \
 && ldconfig

USER ${MY_USER}
# Copy any necessary files into the container (if needed)
# COPY your-local-file /path-in-container

# Set the entrypoint to gmx
# ENTRYPOINT ["/bin/bash"]