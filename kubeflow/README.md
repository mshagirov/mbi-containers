# Kubeflow Dockerfiles
> See https://www.kubeflow.org/docs/components/notebooks/container-images/
> <br/>
> or search for "container images for kubeflow" online.

## Kubeflow Notebook Image Requirements
> From Kubeflow v1.7 documentation
- expose an HTTP interface on port `8888`:
    - kubeflow sets an environment variable `NB_PREFIX` at runtime with the URL path we expect the container be listening under
    - kubeflow uses IFrames, so ensure your application sets
      `Access-Control-Allow-Origin: *` in HTTP response headers
- run as a user called `jovyan`:
    - the home directory of jovyan should be `/home/jovyan`
    - the `UID` of `jovyan` should be `1000`
- start successfully with an empty PVC mounted at `/home/jovyan`:
    - kubeflow mounts a PVC at `/home/jovyan` to keep state across Pod restarts

