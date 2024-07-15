FROM kubeflownotebookswg/jupyter-pytorch-cuda-full:v1.9.0-rc.2

# # args - software versions on BASE
# ARG PYTORCH_VERSION=2.3.0
# ARG TORCHAUDIO_VERSION=2.3.0
# ARG TORCHVISION_VERSION=0.18.0
# ARG PIP_VERSION=24.0

# use pip to install PyG for torch CUDA 12.1
RUN python3 -m pip install --quiet --no-cache-dir \
    torch_geometric==2.5.3
RUN python3 -m pip install --quiet --no-cache-dir \
    pyg_lib torch_scatter torch_sparse torch_cluster torch_spline_conv \
    -f https://data.pyg.org/whl/torch-2.3.0+cu121.html