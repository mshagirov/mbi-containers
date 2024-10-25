FROM kubeflownotebookswg/jupyter-pytorch-cuda-full:v1.9.2

USER root

RUN export DEBIAN_FRONTEND=noninteractive \
 && apt-get -yq update \
 && apt-get -yq install --no-install-recommends \
 aria2 \
 ffmpeg \
 libgfortran5 \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# switch back to the NB_USER
USER 1000
# mamba install overrides cuda jax and jaxlib; re-install them after mamba
RUN mamba install -y -q \
    chex==0.1.86 \
    pdbfixer==1.9 \
    py3dmol==2.4.0 \
    -c conda-forge \
  && mamba clean -a -f -y

RUN python3 -m pip install --quiet --no-cache-dir \
    git+https://github.com/sokrypton/ColabDesign.git --no-deps
# install CUDA packages last:
RUN python3 -m pip install --quiet --no-cache-dir -U \
    biopython \
    dm-haiku \
    dm-tree \
    ffmpeg \
    fsspec \
    immutabledict \
    "jax[cuda12]" \
    joblib \
    ml-collections \
    optax \
    pyrosetta-installer \
    torch \
    torchvision \
    torchaudio
# ENV NB_UID=1000
# ENV USERS_GID=100

USER root
RUN mkdir -pv /content/bindcraft/ \
  && chown -R 1000:100 /content

USER 1000
RUN git clone --depth 1 https://github.com/martinpacesa/BindCraft /content/bindcraft/ \
  && rm -rf /content/bindcraft/.git \
  && chmod +x /content/bindcraft/functions/dssp \
  && chmod +x /content/bindcraft/functions/DAlphaBall.gcc \
  && ln -s /content/bindcraft/ /home/jovyan/bindcraft

RUN python3 -c 'import pyrosetta_installer; pyrosetta_installer.install_pyrosetta()'

