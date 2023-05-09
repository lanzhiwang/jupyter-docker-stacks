# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.


# docker build  --rm --force-rm -t jupyter/tensorflow-notebook:latest ./tensorflow-notebook --build-arg OWNER=jupyter

ARG OWNER=jupyter
# Step 1/6 : ARG OWNER=jupyter

ARG BASE_CONTAINER=$OWNER/scipy-notebook
# Step 2/6 : ARG BASE_CONTAINER=jupyter/scipy-notebook

FROM $BASE_CONTAINER
# Step 3/6 : FROM jupyter/scipy-notebook

LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"
# Step 4/6 : LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"

# Fix: https://github.com/hadolint/hadolint/wiki/DL4006
# Fix: https://github.com/koalaman/shellcheck/wiki/SC3014
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
# Step 5/6 : SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Install Tensorflow
RUN mamba install --yes \
    'tensorflow' && \
    mamba clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"
# Step 6/6 : RUN
# mamba install --yes 'tensorflow' &&
# mamba clean --all -f -y &&
# fix-permissions "/opt/conda" &&
# fix-permissions "/home/jovyan"
