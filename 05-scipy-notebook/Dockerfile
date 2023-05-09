# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.


# docker build  --rm --force-rm -t jupyter/scipy-notebook:latest ./scipy-notebook --build-arg OWNER=jupyter

ARG OWNER=jupyter
# Step 1/15 : ARG OWNER=jupyter

ARG BASE_CONTAINER=$OWNER/minimal-notebook
# Step 2/15 : ARG BASE_CONTAINER=jupyter/minimal-notebook

FROM $BASE_CONTAINER
# Step 3/15 : FROM jupyter/minimal-notebook

LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"
# Step 4/15 : LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"

# Fix: https://github.com/hadolint/hadolint/wiki/DL4006
# Fix: https://github.com/koalaman/shellcheck/wiki/SC3014
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
# Step 5/15 : SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root
# Step 6/15 : USER root

RUN apt-get update --yes && \
    apt-get install --yes --no-install-recommends \
    # for cython: https://cython.readthedocs.io/en/latest/src/quickstart/install.html
    build-essential \
    # for latex labels
    cm-super \
    dvipng \
    # for matplotlib anim
    ffmpeg && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
# Step 7/15 : RUN
# apt-get update --yes &&
# apt-get install --yes --no-install-recommends build-essential cm-super dvipng ffmpeg &&
# apt-get clean &&
# rm -rf /var/lib/apt/lists/*

USER ${NB_UID}
# Step 8/15 : USER 1000

# Install Python 3 packages
RUN mamba install --yes \
    'altair' \
    'beautifulsoup4' \
    'bokeh' \
    'bottleneck' \
    'cloudpickle' \
    'conda-forge::blas=*=openblas' \
    'cython' \
    'dask' \
    'dill' \
    'h5py' \
    'ipympl'\
    'ipywidgets' \
    # Temporary fix for: https://github.com/jupyter/docker-stacks/issues/1851
    'jupyter_server>=2.0.0' \
    'matplotlib-base' \
    'numba' \
    'numexpr' \
    'openpyxl' \
    'pandas' \
    'patsy' \
    'protobuf' \
    'pytables' \
    'scikit-image' \
    'scikit-learn' \
    'scipy' \
    'seaborn' \
    'sqlalchemy' \
    'statsmodels' \
    'sympy' \
    'widgetsnbextension'\
    'xlrd' && \
    mamba clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"
# Step 9/15 : RUN
# mamba install --yes 'altair' 'beautifulsoup4' 'bokeh' 'bottleneck' 'cloudpickle' 'conda-forge::blas=*=openblas' 'cython'
#                     'dask' 'dill' 'h5py' 'ipympl' 'ipywidgets' 'jupyter_server>=2.0.0' 'matplotlib-base' 'numba' 'numexpr'
#                     'openpyxl' 'pandas' 'patsy' 'protobuf' 'pytables' 'scikit-image' 'scikit-learn' 'scipy' 'seaborn'
#                     'sqlalchemy' 'statsmodels' 'sympy' 'widgetsnbextension' 'xlrd' &&
# mamba clean --all -f -y &&
# fix-permissions "/opt/conda" &&
# fix-permissions "/home/jovyan"

# Install facets which does not have a pip or conda package at the moment
WORKDIR /tmp
# Step 10/15 : WORKDIR /tmp

RUN git clone https://github.com/PAIR-code/facets.git && \
    jupyter nbextension install facets/facets-dist/ --sys-prefix && \
    rm -rf /tmp/facets && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"
# Step 11/15 : RUN
# git clone https://github.com/PAIR-code/facets.git &&
# jupyter nbextension install facets/facets-dist/ --sys-prefix &&
# rm -rf /tmp/facets &&
# fix-permissions "/opt/conda" &&
# fix-permissions "/home/jovyan"

# Import matplotlib the first time to build the font cache.
ENV XDG_CACHE_HOME="/home/${NB_USER}/.cache/"
# Step 12/15 : ENV XDG_CACHE_HOME="/home/jovyan/.cache/"

RUN MPLBACKEND=Agg python -c "import matplotlib.pyplot" && \
    fix-permissions "/home/${NB_USER}"
# Step 13/15 : RUN
# MPLBACKEND=Agg python -c "import matplotlib.pyplot" &&
# fix-permissions "/home/jovyan"

USER ${NB_UID}
# Step 14/15 : USER 1000

WORKDIR "${HOME}"
# Step 15/15 : WORKDIR "/home/jovyan"
