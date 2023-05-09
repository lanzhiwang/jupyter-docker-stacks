# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.


# docker build  --rm --force-rm -t jupyter/minimal-notebook:latest ./minimal-notebook --build-arg OWNER=jupyter

ARG OWNER=jupyter
# Step 1/10 : ARG OWNER=jupyter

ARG BASE_CONTAINER=$OWNER/base-notebook
# Step 2/10 : ARG BASE_CONTAINER=jupyter/base-notebook

FROM $BASE_CONTAINER
# Step 3/10 : FROM jupyter/base-notebook

LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"
# Step 4/10 : LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"

# Fix: https://github.com/hadolint/hadolint/wiki/DL4006
# Fix: https://github.com/koalaman/shellcheck/wiki/SC3014
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
# Step 5/10 : SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root
# Step 6/10 : USER root

# Install all OS dependencies for fully functional notebook server
RUN apt-get update --yes && \
    apt-get install --yes --no-install-recommends \
    # Common useful utilities
    git \
    nano-tiny \
    tzdata \
    unzip \
    vim-tiny \
    # git-over-ssh
    openssh-client \
    # less is needed to run help in R
    # see: https://github.com/jupyter/docker-stacks/issues/1588
    less \
    # nbconvert dependencies
    # https://nbconvert.readthedocs.io/en/latest/install.html#installing-tex
    texlive-xetex \
    texlive-fonts-recommended \
    texlive-plain-generic \
    # Enable clipboard on Linux host systems
    xclip && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
# Step 7/10 : RUN
# apt-get update --yes &&
# apt-get install --yes --no-install-recommends git nano-tiny tzdata unzip vim-tiny openssh-client
#                                               less texlive-xetex texlive-fonts-recommended texlive-plain-generic xclip &&
# apt-get clean &&
# rm -rf /var/lib/apt/lists/*

# Create alternative for nano -> nano-tiny
RUN update-alternatives --install /usr/bin/nano nano /bin/nano-tiny 10
# Step 8/10 : RUN update-alternatives --install /usr/bin/nano nano /bin/nano-tiny 10

# Switch back to jovyan to avoid accidental container runs as root
USER ${NB_UID}
# Step 9/10 : USER 1000

# Add R mimetype option to specify how the plot returns from R to the browser
COPY --chown=1000:100 Rprofile.site /opt/conda/lib/R/etc/
# Step 10/10 : COPY --chown=1000:100 Rprofile.site /opt/conda/lib/R/etc/
