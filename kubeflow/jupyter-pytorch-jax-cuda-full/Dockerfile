FROM kubeflownotebookswg/jupyter-pytorch-cuda-full:v1.9.2

USER root

RUN export DEBIAN_FRONTEND=noninteractive \
 && apt-get -yq update \
 && apt-get -yq install --no-install-recommends aria2 \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# switch back to the NB_USER
USER 1000

RUN python3 -m pip install --quiet --no-cache-dir -U \
    dm-haiku \
    dm-tree \
    ffmpeg \
    fsspec \
    immutabledict \
    "jax[cuda12]" \
    joblib \
    ml-collections \
    optax \
    torch \
    torchvision \
    torchaudio

# mamba seems to override cuda versions of jax and jaxlib; re-install them after mamba
RUN mamba install -y -q \
    chex==0.1.86 \
    # dm-haiku \
    # dm-tree \
    # ffmpeg \
    # fsspec \
    # immutabledict \
    # joblib \
    # ml-collections \
    # optax \
    pdbfixer==1.9 \
    py3dmol==2.4.0 \
    -c conda-forge \
    # -c anaconda -c nvidia  \
  && mamba install -y -q pyrosetta --channel https://conda.graylab.jhu.edu \
  && mamba clean -a -f -y

RUN python3 -m pip install --quiet --no-cache-dir \
    git+https://github.com/sokrypton/ColabDesign.git --no-deps

