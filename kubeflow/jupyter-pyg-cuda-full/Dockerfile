FROM kubeflownotebookswg/jupyter-pytorch-cuda-full:v1.9.0

# # args - software versions on BASE
# ARG PYTORCH_VERSION=2.3.0
# ARG TORCHAUDIO_VERSION=2.3.0
# ARG TORCHVISION_VERSION=0.18.0
# ARG PIP_VERSION=24.0

USER root

RUN export DEBIAN_FRONTEND=noninteractive \
 && apt-get -yq update \
 && apt-get -yq install --no-install-recommends \
  openssh-client \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# set used back to jovyan:default
USER 1000
# use pip to install PyG for torch CUDA 12.1
RUN python3 -m pip install --quiet --no-cache-dir \
    torch_geometric==2.6.1
RUN python3 -m pip install --quiet --no-cache-dir \
    pyg_lib \
    torch_cluster \
    torch_scatter \
    torch_sparse \
    torch_spline_conv \
    -f https://data.pyg.org/whl/torch-2.3.0+cu121.html

