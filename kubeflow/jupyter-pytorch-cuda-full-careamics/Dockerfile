FROM muratshagirov/jupyter-pytorch-cu121:latest

USER $NB_UID

# install - requirements.txt
COPY --chown=${NB_USER}:${NB_GID} requirements-careamics.txt /tmp/requirements.txt
RUN python3 -m pip install -r /tmp/requirements.txt --quiet --no-cache-dir \
 && rm -f /tmp/requirements.txt
