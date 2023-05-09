# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.


# docker build  --rm --force-rm -t jupyter/all-spark-notebook:latest ./all-spark-notebook --build-arg OWNER=jupyter

ARG OWNER=jupyter
# Step 1/11 : ARG OWNER=jupyter

ARG BASE_CONTAINER=$OWNER/pyspark-notebook
# Step 2/11 : ARG BASE_CONTAINER=jupyter/pyspark-notebook

FROM $BASE_CONTAINER
# Step 3/11 : FROM jupyter/pyspark-notebook

LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"
# Step 4/11 : LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"

# Fix: https://github.com/hadolint/hadolint/wiki/DL4006
# Fix: https://github.com/koalaman/shellcheck/wiki/SC3014
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
# Step 5/11 : SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root
# Step 6/11 : USER root

# RSpark config
ENV R_LIBS_USER "${SPARK_HOME}/R/lib"
# Step 7/11 : ENV R_LIBS_USER "/usr/local/spark/R/lib"

RUN fix-permissions "${R_LIBS_USER}"
# Step 8/11 : RUN fix-permissions /usr/local/spark/R/lib

# R pre-requisites
RUN apt-get update --yes && \
    apt-get install --yes --no-install-recommends \
    fonts-dejavu \
    gfortran \
    gcc && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
# Step 9/11 : RUN
# apt-get update --yes &&
# apt-get install --yes --no-install-recommends fonts-dejavu gfortran gcc &&
# apt-get clean &&
# rm -rf /var/lib/apt/lists/*

USER ${NB_UID}
# Step 9/10 : USER 1000

# R packages including IRKernel which gets installed globally.
RUN mamba install --yes \
    'r-base' \
    'r-ggplot2' \
    'r-irkernel' \
    'r-rcurl' \
    'r-sparklyr' && \
    mamba clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"
Step 10/10 : RUN
mamba install --yes 'r-base' 'r-ggplot2' 'r-irkernel' 'r-rcurl' 'r-sparklyr' &&
mamba clean --all -f -y &&
fix-permissions "/opt/conda" &&
fix-permissions "/home/jovyan"
