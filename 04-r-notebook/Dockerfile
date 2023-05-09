# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.


# docker build  --rm --force-rm -t jupyter/r-notebook:latest ./r-notebook --build-arg OWNER=jupyter

ARG OWNER=jupyter
# Step 1/9 : ARG OWNER=jupyter

ARG BASE_CONTAINER=$OWNER/minimal-notebook
# Step 2/9 : ARG BASE_CONTAINER=jupyter/minimal-notebook

FROM $BASE_CONTAINER
# Step 3/9 : FROM jupyter/minimal-notebook

LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"
# Step 4/9 : LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"

# Fix: https://github.com/hadolint/hadolint/wiki/DL4006
# Fix: https://github.com/koalaman/shellcheck/wiki/SC3014
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
# Step 5/9 : SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root
# Step 6/9 : USER root

# R pre-requisites
RUN apt-get update --yes && \
    apt-get install --yes --no-install-recommends \
    fonts-dejavu \
    unixodbc \
    unixodbc-dev \
    r-cran-rodbc \
    gfortran \
    gcc && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
# Step 7/9 : RUN
# apt-get update --yes &&
# apt-get install --yes --no-install-recommends fonts-dejavu unixodbc unixodbc-dev r-cran-rodbc gfortran gcc &&
# apt-get clean &&
# rm -rf /var/lib/apt/lists/*

USER ${NB_UID}
# Step 8/9 : USER 1000

# R packages including IRKernel which gets installed globally.
# r-e1071: dependency of the caret R package
RUN mamba install --yes \
    'r-base' \
    'r-caret' \
    'r-crayon' \
    'r-devtools' \
    'r-e1071' \
    'r-forecast' \
    'r-hexbin' \
    'r-htmltools' \
    'r-htmlwidgets' \
    'r-irkernel' \
    'r-nycflights13' \
    'r-randomforest' \
    'r-rcurl' \
    'r-rmarkdown' \
    'r-rodbc' \
    'r-rsqlite' \
    'r-shiny' \
    'r-tidymodels' \
    'r-tidyverse' \
    'unixodbc' && \
    mamba clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"
# Step 9/9 : RUN
# mamba install --yes 'r-base' 'r-caret' 'r-crayon' 'r-devtools' 'r-e1071' 'r-forecast' 'r-hexbin' 'r-htmltools' 'r-htmlwidgets'
#                     'r-irkernel' 'r-nycflights13' 'r-randomforest' 'r-rcurl' 'r-rmarkdown' 'r-rodbc' 'r-rsqlite' 'r-shiny'
#                     'r-tidymodels' 'r-tidyverse' 'unixodbc' &&
# mamba clean --all -f -y &&
# fix-permissions "/opt/conda" &&
# fix-permissions "/home/jovyan"
