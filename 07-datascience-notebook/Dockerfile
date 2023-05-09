# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.


# docker build  --rm --force-rm -t jupyter/datascience-notebook:latest ./datascience-notebook --build-arg OWNER=jupyter

ARG OWNER=jupyter
# Step 1/16 : ARG OWNER=jupyter

ARG BASE_CONTAINER=$OWNER/scipy-notebook
# Step 2/16 : ARG BASE_CONTAINER=jupyter/scipy-notebook

FROM $BASE_CONTAINER
# Step 3/16 : FROM jupyter/scipy-notebook

LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"
# Step 4/16 : LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"

# Fix: https://github.com/hadolint/hadolint/wiki/DL4006
# Fix: https://github.com/koalaman/shellcheck/wiki/SC3014
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
# Step 5/16 : SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root
# Step 6/16 : USER root

# Julia installation
# Default values can be overridden at build time
# (ARGS are in lower case to distinguish them from ENV)
# Check https://julialang.org/downloads/
ARG julia_version="1.8.5"
# Step 7/16 : ARG julia_version="1.8.5"

# R pre-requisites
RUN apt-get update --yes && \
    apt-get install --yes --no-install-recommends \
    fonts-dejavu \
    gfortran \
    gcc && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
# Step 8/16 : RUN
# apt-get update --yes &&
# apt-get install --yes --no-install-recommends fonts-dejavu gfortran gcc &&
# apt-get clean &&
# rm -rf /var/lib/apt/lists/*

# Julia dependencies
# install Julia packages in /opt/julia instead of ${HOME}
ENV JULIA_DEPOT_PATH=/opt/julia \
    JULIA_PKGDIR=/opt/julia \
    JULIA_VERSION="${julia_version}"
# Step 9/16 : ENV
# JULIA_DEPOT_PATH=/opt/julia
# JULIA_PKGDIR=/opt/julia
# JULIA_VERSION="1.8.5"

WORKDIR /tmp
# Step 10/16 : WORKDIR /tmp

# hadolint ignore=SC2046
RUN set -x && \
    julia_arch=$(uname -m) && \
    julia_short_arch="${julia_arch}" && \
    if [ "${julia_short_arch}" == "x86_64" ]; then \
      julia_short_arch="x64"; \
    fi; \
    julia_installer="julia-${JULIA_VERSION}-linux-${julia_arch}.tar.gz" && \
    julia_major_minor=$(echo "${JULIA_VERSION}" | cut -d. -f 1,2) && \
    mkdir "/opt/julia-${JULIA_VERSION}" && \
    wget -q "https://julialang-s3.julialang.org/bin/linux/${julia_short_arch}/${julia_major_minor}/${julia_installer}" && \
    tar xzf "${julia_installer}" -C "/opt/julia-${JULIA_VERSION}" --strip-components=1 && \
    rm "${julia_installer}" && \
    ln -fs /opt/julia-*/bin/julia /usr/local/bin/julia
# Step 11/16 : RUN
# set -x &&
# julia_arch=$(uname -m) &&
# julia_short_arch="${julia_arch}" &&
# if [ "${julia_short_arch}" == "x86_64" ]; then
#     julia_short_arch="x64";
# fi;
# julia_installer="julia-1.8.5-linux-x86_64.tar.gz" &&
# julia_major_minor=$(echo "1.8.5" | cut -d. -f 1,2) &&
# mkdir "/opt/julia-1.8.5" &&
# wget -q "https://julialang-s3.julialang.org/bin/linux/x64/1.8/julia-1.8.5-linux-x86_64.tar.gz" &&
# tar xzf "julia-1.8.5-linux-x86_64.tar.gz" -C "/opt/julia-1.8.5" --strip-components=1 &&
# rm "julia-1.8.5-linux-x86_64.tar.gz" &&
# ln -fs /opt/julia-*/bin/julia /usr/local/bin/julia

# Show Julia where conda libraries are \
RUN mkdir /etc/julia && \
    echo "push!(Libdl.DL_LOAD_PATH, \"${CONDA_DIR}/lib\")" >> /etc/julia/juliarc.jl && \
    # Create JULIA_PKGDIR \
    mkdir "${JULIA_PKGDIR}" && \
    chown "${NB_USER}" "${JULIA_PKGDIR}" && \
    fix-permissions "${JULIA_PKGDIR}"
# Step 12/16 : RUN
# mkdir /etc/julia &&
# echo "push!(Libdl.DL_LOAD_PATH, \"/opt/conda/lib\")" >> /etc/julia/juliarc.jl &&
# mkdir "/opt/julia" &&
# chown "${NB_USER}" "/opt/julia" &&
# fix-permissions "/opt/julia"

USER ${NB_UID}
# Step 13/16 : USER 1000

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
    'rpy2' \
    'unixodbc' && \
    mamba clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"
# Step 14/16 : RUN
# mamba install --yes 'r-base' 'r-caret' 'r-crayon' 'r-devtools' 'r-e1071' 'r-forecast' 'r-hexbin' 'r-htmltools'
#                     'r-htmlwidgets' 'r-irkernel' 'r-nycflights13' 'r-randomforest' 'r-rcurl' 'r-rmarkdown' 'r-rodbc'
#                     'r-rsqlite' 'r-shiny' 'r-tidymodels' 'r-tidyverse' 'rpy2' 'unixodbc' &&
# mamba clean --all -f -y &&
# fix-permissions "/opt/conda" &&
# fix-permissions "/home/jovyan"

# Add Julia packages.
# Install IJulia as jovyan and then move the kernelspec out
# to the system share location. Avoids problems with runtime UID change not
# taking effect properly on the .local folder in the jovyan home dir.
RUN julia -e 'import Pkg; Pkg.update()' && \
    julia -e 'import Pkg; Pkg.add("HDF5")' && \
    julia -e 'using Pkg; pkg"add IJulia"; pkg"precompile"' && \
    # move kernelspec out of home \
    mv "${HOME}/.local/share/jupyter/kernels/julia"* "${CONDA_DIR}/share/jupyter/kernels/" && \
    chmod -R go+rx "${CONDA_DIR}/share/jupyter" && \
    rm -rf "${HOME}/.local" && \
    fix-permissions "${JULIA_PKGDIR}" "${CONDA_DIR}/share/jupyter"
# Step 15/16 : RUN
# julia -e 'import Pkg; Pkg.update()' &&
# julia -e 'import Pkg; Pkg.add("HDF5")' &&
# julia -e 'using Pkg; pkg"add IJulia"; pkg"precompile"' &&
# mv "/home/jovyan/.local/share/jupyter/kernels/julia"* "/opt/conda/share/jupyter/kernels/" &&
# chmod -R go+rx "/opt/conda/share/jupyter" &&
# rm -rf "/home/jovyan/.local" &&
# fix-permissions "${JULIA_PKGDIR}" "/opt/conda/share/jupyter"

WORKDIR "${HOME}"
# Step 15/16 : WORKDIR /home/jovyan
