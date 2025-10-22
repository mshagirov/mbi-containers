FROM ghcr.io/kubeflow/kubeflow/notebook-servers/jupyter:v1.9.0

USER root

RUN export DEBIAN_FRONTEND=noninteractive \
 && apt-get -yq update \
 && apt-get -yq install --no-install-recommends \
  build-essential \
  libomp-dev \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

USER 1000

COPY --chown=${NB_USER}:${NB_GID} ./bash_profile.sh ${HOME_TMP}/.bash_profile

COPY --chown=${NB_USER}:${NB_GID} ./PolyHoop ${HOME_TMP}/polyhoop

RUN mamba install -y -q \
  bokeh==3.8.0 \
  cloudpickle==3.1.1 \
  dill==0.4.0 \
  ipympl==0.9.8 \
  jupyterview==0.7.0 \
  matplotlib==3.10.7 \
  numpy==1.26.4 \
  pandas==2.3.3 \
  scikit-image==0.25.2 \
  scipy==1.16.2 \
  seaborn==0.13.2 \
&& mamba clean -a -f -y

COPY --chown=${NB_USER}:${NB_GID} ./requirements.txt /tmp/requirements.txt
RUN python3 -m pip install -r /tmp/requirements.txt --quiet --no-cache-dir \
 && rm -f /tmp/requirements.txt
