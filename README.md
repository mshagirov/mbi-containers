# Dockerfiles and Source Code for MBI Containers
> Murat Shagirov

# Kubeflow Containers
> Follows KF naming and versioning convention <br/>
> where possible, the images use KF base images.
- Notebook base:
  - Jupyter notebook image names start with `jupyter-*`.
  - VS Code (notebook) image names start with `codeserver-*`
- CPU and CUDA images:
  - CUDA images contain `*-cuda-*` in their names, otherwise they are CPU container images.
- Dockerfiles location:
  - [./kubeflow](./kubeflow)
