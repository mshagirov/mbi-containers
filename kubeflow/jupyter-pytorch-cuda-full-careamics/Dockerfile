FROM muratshagirov/jupyter-pytorch-cu121@sha256:5cd15eb9a17ffb4753cdca03e02477b103cf4a50dfb409e4f92a47f7dc289988

USER $NB_UID

# install - requirements.txt
COPY --chown=${NB_USER}:${NB_GID} requirements-careamics.txt /tmp/requirements.txt
RUN python3 -m pip install -r /tmp/requirements.txt --quiet --no-cache-dir \
 && rm -f /tmp/requirements.txt
