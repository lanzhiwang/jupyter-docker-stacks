# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.

# Ubuntu 22.04 (jammy)
# https://hub.docker.com/_/ubuntu/tags?page=1&name=jammy


# docker build --rm --force-rm -t jupyter/docker-stacks-foundation:latest ./docker-stacks-foundation --build-arg OWNER=jupyter

ARG ROOT_CONTAINER=ubuntu:22.04
# Step 1/27 : ARG ROOT_CONTAINER=ubuntu:22.04

FROM $ROOT_CONTAINER
# Step 2/27 : FROM $ROOT_CONTAINER
# Step 2/27 : FROM ubuntu:22.04

LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"
# Step 3/27 : LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"

ARG NB_USER="jovyan"
# Step 4/27 : ARG NB_USER="jovyan"

ARG NB_UID="1000"
# Step 5/27 : ARG NB_UID="1000"

ARG NB_GID="100"
# Step 6/27 : ARG NB_GID="100"

# Fix: https://github.com/hadolint/hadolint/wiki/DL4006
# Fix: https://github.com/koalaman/shellcheck/wiki/SC3014
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
# Step 7/27 : SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root
# Step 8/27 : USER root

# Install all OS dependencies for notebook server that starts but lacks all
# features (e.g., download as all possible file formats)
ENV DEBIAN_FRONTEND noninteractive
# Step 9/27 : ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update --yes && \
    # - apt-get upgrade is run to patch known vulnerabilities in apt-get packages as
    #   the ubuntu base image is rebuilt too seldom sometimes (less than once a month)
    apt-get upgrade --yes && \
    apt-get install --yes --no-install-recommends \
    # - bzip2 is necessary to extract the micromamba executable.
    bzip2 \
    ca-certificates \
    locales \
    sudo \
    # - tini is installed as a helpful container entrypoint that reaps zombie
    #   processes and such of the actual executable we want to start, see
    #   https://github.com/krallin/tini#why-tini for details.
    tini \
    wget && \
    apt-get clean && rm -rf /var/lib/apt/lists/* && \
    echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen
# Step 10/27 : RUN
# apt-get update --yes &&
# apt-get upgrade --yes &&
# apt-get install --yes --no-install-recommends bzip2 ca-certificates locales sudo tini wget &&
# apt-get clean &&
# rm -rf /var/lib/apt/lists/* &&
# echo "en_US.UTF-8 UTF-8" > /etc/locale.gen &&
# locale-gen

# Configure environment
ENV CONDA_DIR=/opt/conda \
    SHELL=/bin/bash \
    NB_USER="${NB_USER}" \
    NB_UID=${NB_UID} \
    NB_GID=${NB_GID} \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8
# Step 11/27 : ENV
# CONDA_DIR=/opt/conda
# SHELL=/bin/bash
# NB_USER="jovyan"
# NB_UID=1000
# NB_GID=100
# LC_ALL=en_US.UTF-8
# LANG=en_US.UTF-8
# LANGUAGE=en_US.UTF-8

ENV PATH="${CONDA_DIR}/bin:${PATH}" \
    HOME="/home/${NB_USER}"
# Step 12/27 : ENV
# PATH="/opt/conda/bin:${PATH}"
# HOME="/home/jovyan"

# Copy a script that we will use to correct permissions after running certain commands
COPY fix-permissions /usr/local/bin/fix-permissions
# Step 13/27 : COPY fix-permissions /usr/local/bin/fix-permissions

RUN chmod a+rx /usr/local/bin/fix-permissions
# Step 14/27 : RUN chmod a+rx /usr/local/bin/fix-permissions

# Enable prompt color in the skeleton .bashrc before creating the default NB_USER
# hadolint ignore=SC2016
RUN sed -i 's/^#force_color_prompt=yes/force_color_prompt=yes/' /etc/skel/.bashrc && \
   # Add call to conda init script see https://stackoverflow.com/a/58081608/4413446
   echo 'eval "$(command conda shell.bash hook 2> /dev/null)"' >> /etc/skel/.bashrc
# Step 15/27 : RUN
# sed -i 's/^#force_color_prompt=yes/force_color_prompt=yes/' /etc/skel/.bashrc &&
# echo 'eval "$(command conda shell.bash hook 2> /dev/null)"' >> /etc/skel/.bashrc

# Create NB_USER with name jovyan user with UID=1000 and in the 'users' group
# and make sure these dirs are writable by the `users` group.
RUN echo "auth requisite pam_deny.so" >> /etc/pam.d/su && \
    sed -i.bak -e 's/^%admin/#%admin/' /etc/sudoers && \
    sed -i.bak -e 's/^%sudo/#%sudo/' /etc/sudoers && \
    useradd -l -m -s /bin/bash -N -u "${NB_UID}" "${NB_USER}" && \
    mkdir -p "${CONDA_DIR}" && \
    chown "${NB_USER}:${NB_GID}" "${CONDA_DIR}" && \
    chmod g+w /etc/passwd && \
    fix-permissions "${HOME}" && \
    fix-permissions "${CONDA_DIR}"
# Step 16/27 : RUN
# echo "auth requisite pam_deny.so" >> /etc/pam.d/su &&
# sed -i.bak -e 's/^%admin/#%admin/' /etc/sudoers &&
# sed -i.bak -e 's/^%sudo/#%sudo/' /etc/sudoers &&
# useradd -l -m -s /bin/bash -N -u "1000" "jovyan" &&
# mkdir -p "/opt/conda" &&
# chown "jovyan:100" "/opt/conda" &&
# chmod g+w /etc/passwd &&
# fix-permissions "/home/jovyan" &&
# fix-permissions "/opt/conda"

USER ${NB_UID}
# Step 17/27 : USER 1000

# Pin python version here, or set it to "default"
ARG PYTHON_VERSION=3.10
# Step 18/27 : ARG PYTHON_VERSION=3.10

# Setup work directory for backward-compatibility
RUN mkdir "/home/${NB_USER}/work" && \
    fix-permissions "/home/${NB_USER}"
# Step 19/27 : RUN
# mkdir "/home/jovyan/work" &&
# fix-permissions "/home/jovyan"

# Download and install Micromamba, and initialize Conda prefix.
#   <https://github.com/mamba-org/mamba#micromamba>
#   Similar projects using Micromamba:
#     - Micromamba-Docker: <https://github.com/mamba-org/micromamba-docker>
#     - repo2docker: <https://github.com/jupyterhub/repo2docker>
# Install Python, Mamba and jupyter_core
# Cleanup temporary files and remove Micromamba
# Correct permissions
# Do all this in a single RUN command to avoid duplicating all of the
# files across image layers when the permissions change
COPY --chown="${NB_UID}:${NB_GID}" initial-condarc "${CONDA_DIR}/.condarc"
# Step 20/27 : COPY --chown="1000:100" initial-condarc "/opt/conda/.condarc"

WORKDIR /tmp
# Step 21/27 : WORKDIR /tmp

RUN set -x && \
    arch=$(uname -m) && \
    if [ "${arch}" = "x86_64" ]; then \
        # Should be simpler, see <https://github.com/mamba-org/mamba/issues/1437>
        arch="64"; \
    fi && \
    wget -qO /tmp/micromamba.tar.bz2 \
        "https://micromamba.snakepit.net/api/micromamba/linux-${arch}/latest" && \
    tar -xvjf /tmp/micromamba.tar.bz2 --strip-components=1 bin/micromamba && \
    rm /tmp/micromamba.tar.bz2 && \
    PYTHON_SPECIFIER="python=${PYTHON_VERSION}" && \
    if [[ "${PYTHON_VERSION}" == "default" ]]; then PYTHON_SPECIFIER="python"; fi && \
    # Install the packages
    ./micromamba install \
        --root-prefix="${CONDA_DIR}" \
        --prefix="${CONDA_DIR}" \
        --yes \
        "${PYTHON_SPECIFIER}" \
        'mamba' \
        'jupyter_core' && \
    rm micromamba && \
    # Pin major.minor version of python
    mamba list python | grep '^python ' | tr -s ' ' | cut -d ' ' -f 1,2 >> "${CONDA_DIR}/conda-meta/pinned" && \
    mamba clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"
# Step 22/27 : RUN
# set -x &&
# arch=$(uname -m) &&
# if [ "${arch}" = "x86_64" ]; then
#     arch="64";
# fi &&
# wget -qO /tmp/micromamba.tar.bz2 "https://micromamba.snakepit.net/api/micromamba/linux-64/latest" &&
# tar -xvjf /tmp/micromamba.tar.bz2 --strip-components=1 bin/micromamba &&
# rm /tmp/micromamba.tar.bz2 &&
# PYTHON_SPECIFIER="python=3.10" &&
# if [[ "3.10" == "default" ]]; then
#     PYTHON_SPECIFIER="python";
# fi &&
# ./micromamba install --root-prefix="/opt/conda" --prefix="/opt/conda" --yes "python=3.10" 'mamba' 'jupyter_core' &&
# rm micromamba &&
# mamba list python | grep '^python ' | tr -s ' ' | cut -d ' ' -f 1,2 >> "/opt/conda/conda-meta/pinned" &&
# mamba clean --all -f -y &&
# fix-permissions "/opt/conda" &&
# fix-permissions "/home/jovyan"

# ++ uname -m
# + arch=x86_64
# + '[' x86_64 = x86_64 ']'
# + arch=64
# + wget -qO /tmp/micromamba.tar.bz2 https://micromamba.snakepit.net/api/micromamba/linux-64/latest
# + tar -xvjf /tmp/micromamba.tar.bz2 --strip-components=1 bin/micromamba
# bin/micromamba
# $ ll
# total 10312
# drwxr-xr-x   6 huzhi  staff      192  3 29 18:09 ./
# drwxr-xr-x  11 huzhi  staff      352  3 29 17:28 ../
# drwxr-xr-x   3 huzhi  staff       96  3 29 18:09 bin/
# drwxr-xr-x  12 huzhi  staff      384  3 29 18:09 info/
# -rw-r--r--@  1 huzhi  staff  5173934  3 29 18:09 micromamba-1.4.1-0.tar.bz2
# $ ll bin
# total 26648
# drwxr-xr-x  3 huzhi  staff        96  3 29 18:09 ./
# drwxr-xr-x  6 huzhi  staff       192  3 29 18:09 ../
# -rwxr-xr-x@ 1 huzhi  staff  13639792  3 28 19:32 micromamba*
# $ ll info
# total 56
# drwxr-xr-x  12 huzhi  staff   384  3 29 18:09 ./
# drwxr-xr-x   6 huzhi  staff   192  3 29 18:09 ../
# -rw-r--r--@  1 huzhi  staff  6855  3 28 19:32 about.json
# -rw-r--r--@  1 huzhi  staff    15  3 28 19:32 files
# -rw-r--r--@  1 huzhi  staff     0  3 28 19:32 git
# -rw-r--r--@  1 huzhi  staff   278  3 28 19:32 has_prefix
# -rw-r--r--@  1 huzhi  staff   256  3 28 19:32 hash_input.json
# -rw-r--r--@  1 huzhi  staff   310  3 28 19:32 index.json
# drwxr-xr-x  20 huzhi  staff   640  3 29 18:09 licenses/
# -rw-r--r--@  1 huzhi  staff   550  3 28 19:32 paths.json
# drwxr-xr-x  29 huzhi  staff   928  3 29 18:09 recipe/
# drwxr-xr-x   3 huzhi  staff    96  3 29 18:09 test/
# + rm /tmp/micromamba.tar.bz2
# + PYTHON_SPECIFIER=python=3.10
# + [[ 3.10 == \d\e\f\a\u\l\t ]]
# + ./micromamba install --root-prefix=/opt/conda --prefix=/opt/conda --yes python=3.10 mamba jupyter_core

#                                            __
#           __  ______ ___  ____ _____ ___  / /_  ____ _
#          / / / / __ `__ \/ __ `/ __ `__ \/ __ \/ __ `/
#         / /_/ / / / / / / /_/ / / / / / / /_/ / /_/ /
#        / .___/_/ /_/ /_/\__,_/_/ /_/ /_/_.___/\__,_/
#       /_/


# Transaction

#   Prefix: /opt/conda

#   Updating specs:

#    - python=3.10
#    - mamba
#    - jupyter_core


#   Package                         Version  Build               Channel                   Size
# ───────────────────────────────────────────────────────────────────────────────────────────────
#   Install:
# ───────────────────────────────────────────────────────────────────────────────────────────────

#   + _libgcc_mutex                     0.1  conda_forge         conda-forge/linux-64       3kB
#   + _openmp_mutex                     4.5  2_gnu               conda-forge/linux-64      24kB
#   + boltons                        23.0.0  pyhd8ed1ab_0        conda-forge/noarch       303kB
#   + brotlipy                        0.7.0  py310h5764c6d_1005  conda-forge/linux-64     351kB
#   + bzip2                           1.0.8  h7f98852_4          conda-forge/linux-64     496kB
#   + c-ares                         1.18.1  h7f98852_0          conda-forge/linux-64     115kB
#   + ca-certificates             2022.12.7  ha878542_0          conda-forge/linux-64     146kB
#   + certifi                     2022.12.7  pyhd8ed1ab_0        conda-forge/noarch       151kB
#   + cffi                           1.15.1  py310h255011f_3     conda-forge/linux-64     237kB
#   + charset-normalizer              2.1.1  pyhd8ed1ab_0        conda-forge/noarch        36kB
#   + colorama                        0.4.6  pyhd8ed1ab_0        conda-forge/noarch        25kB
#   + conda                          23.3.0  py310hff52083_0     conda-forge/linux-64     953kB
#   + conda-package-handling          2.0.2  pyh38be061_0        conda-forge/noarch       253kB
#   + conda-package-streaming         0.7.0  pyhd8ed1ab_1        conda-forge/noarch        17kB
#   + cryptography                   40.0.1  py310h34c0648_0     conda-forge/linux-64       2MB
#   + fmt                             9.1.0  h924138e_0          conda-forge/linux-64     190kB
#   + icu                              72.1  hcb278e6_0          conda-forge/linux-64      12MB
#   + idna                              3.4  pyhd8ed1ab_0        conda-forge/noarch        57kB
#   + jupyter_core                    5.3.0  py310hff52083_0     conda-forge/linux-64      91kB
#   + keyutils                        1.6.1  h166bdaf_0          conda-forge/linux-64     118kB
#   + krb5                           1.20.1  h81ceb04_0          conda-forge/linux-64       1MB
#   + ld_impl_linux-64                 2.40  h41732ed_0          conda-forge/linux-64     705kB
#   + libarchive                      3.6.2  h3d51595_0          conda-forge/linux-64     836kB
#   + libcurl                        7.88.1  hdc1c0ab_1          conda-forge/linux-64     359kB
#   + libedit                  3.1.20191231  he28a2e2_2          conda-forge/linux-64     124kB
#   + libev                            4.33  h516909a_1          conda-forge/linux-64     106kB
#   + libffi                          3.4.2  h7f98852_5          conda-forge/linux-64      58kB
#   + libgcc-ng                      12.2.0  h65d4601_19         conda-forge/linux-64     954kB
#   + libgomp                        12.2.0  h65d4601_19         conda-forge/linux-64     466kB
#   + libiconv                         1.17  h166bdaf_0          conda-forge/linux-64       1MB
#   + libmamba                        1.4.1  hcea66bb_0          conda-forge/linux-64       1MB
#   + libmambapy                      1.4.1  py310h1428755_0     conda-forge/linux-64     268kB
#   + libnghttp2                     1.52.0  h61bc06f_0          conda-forge/linux-64     622kB
#   + libnsl                          2.0.0  h7f98852_0          conda-forge/linux-64      31kB
#   + libsolv                        0.7.23  h3eb15da_0          conda-forge/linux-64     462kB
#   + libsqlite                      3.40.0  h753d276_0          conda-forge/linux-64     810kB
#   + libssh2                        1.10.0  hf14f497_3          conda-forge/linux-64     239kB
#   + libstdcxx-ng                   12.2.0  h46fd767_19         conda-forge/linux-64       4MB
#   + libuuid                        2.32.1  h7f98852_1000       conda-forge/linux-64      28kB
#   + libxml2                        2.10.3  hfdac1af_6          conda-forge/linux-64     714kB
#   + libzlib                        1.2.13  h166bdaf_4          conda-forge/linux-64      66kB
#   + lz4-c                           1.9.4  hcb278e6_0          conda-forge/linux-64     143kB
#   + lzo                              2.10  h516909a_1000       conda-forge/linux-64     321kB
#   + mamba                           1.4.1  py310h51d5547_0     conda-forge/linux-64      51kB
#   + ncurses                           6.3  h27087fc_1          conda-forge/linux-64       1MB
#   + openssl                         3.1.0  h0b41bf4_0          conda-forge/linux-64       3MB
#   + packaging                        23.0  pyhd8ed1ab_0        conda-forge/noarch        41kB
#   + pip                            23.0.1  pyhd8ed1ab_0        conda-forge/noarch         1MB
#   + platformdirs                    3.2.0  pyhd8ed1ab_0        conda-forge/noarch        18kB
#   + pluggy                          1.0.0  pyhd8ed1ab_5        conda-forge/noarch        16kB
#   + pybind11-abi                        4  hd8ed1ab_3          conda-forge/noarch        10kB
#   + pycosat                         0.6.4  py310h5764c6d_1     conda-forge/linux-64     111kB
#   + pycparser                        2.21  pyhd8ed1ab_0        conda-forge/noarch       103kB
#   + pyopenssl                      23.1.1  pyhd8ed1ab_0        conda-forge/noarch       128kB
#   + pysocks                         1.7.1  pyha2e5f31_6        conda-forge/noarch        19kB
#   + python                        3.10.10  he550d4f_0_cpython  conda-forge/linux-64      26MB
#   + python_abi                       3.10  3_cp310             conda-forge/linux-64       6kB
#   + readline                          8.2  h8228510_1          conda-forge/linux-64     281kB
#   + reproc                         14.2.4  h0b41bf4_0          conda-forge/linux-64      30kB
#   + reproc-cpp                     14.2.4  hcb278e6_0          conda-forge/linux-64      22kB
#   + requests                       2.28.2  pyhd8ed1ab_0        conda-forge/noarch        57kB
#   + ruamel.yaml                   0.17.21  py310h1fa729e_3     conda-forge/linux-64     190kB
#   + ruamel.yaml.clib                0.2.7  py310h1fa729e_1     conda-forge/linux-64     135kB
#   + setuptools                     67.6.1  pyhd8ed1ab_0        conda-forge/noarch       580kB
#   + tk                             8.6.12  h27826a3_0          conda-forge/linux-64       3MB
#   + toolz                          0.12.0  pyhd8ed1ab_0        conda-forge/noarch        49kB
#   + tqdm                           4.65.0  pyhd8ed1ab_1        conda-forge/noarch        88kB
#   + traitlets                       5.9.0  pyhd8ed1ab_0        conda-forge/noarch        98kB
#   + typing-extensions               4.5.0  hd8ed1ab_0          conda-forge/noarch        10kB
#   + typing_extensions               4.5.0  pyha770c72_0        conda-forge/noarch        31kB
#   + tzdata                          2023c  h71feb2d_0          conda-forge/noarch       118kB
#   + urllib3                       1.26.15  pyhd8ed1ab_0        conda-forge/noarch       113kB
#   + wheel                          0.40.0  pyhd8ed1ab_0        conda-forge/noarch        56kB
#   + xz                              5.2.6  h166bdaf_0          conda-forge/linux-64     418kB
#   + yaml-cpp                        0.7.0  h27087fc_2          conda-forge/linux-64     220kB
#   + zstandard                      0.19.0  py310hdeb6495_1     conda-forge/linux-64     382kB
#   + zstd                            1.5.2  h3eb15da_6          conda-forge/linux-64     420kB

#   Summary:

#   Install: 77 packages

#   Total download: 71MB

# ───────────────────────────────────────────────────────────────────────────────────────────────



# Transaction starting
# Linking _libgcc_mutex-0.1-conda_forge
# Linking ld_impl_linux-64-2.40-h41732ed_0
# Linking ca-certificates-2022.12.7-ha878542_0
# Linking python_abi-3.10-3_cp310
# Linking libstdcxx-ng-12.2.0-h46fd767_19
# Linking libgomp-12.2.0-h65d4601_19
# Linking _openmp_mutex-4.5-2_gnu
# Linking libgcc-ng-12.2.0-h65d4601_19
# Linking lzo-2.10-h516909a_1000
# Linking libuuid-2.32.1-h7f98852_1000
# Linking libev-4.33-h516909a_1
# Linking lz4-c-1.9.4-hcb278e6_0
# Linking icu-72.1-hcb278e6_0
# Linking libiconv-1.17-h166bdaf_0
# Linking c-ares-1.18.1-h7f98852_0
# Linking keyutils-1.6.1-h166bdaf_0
# Linking reproc-14.2.4-h0b41bf4_0
# Linking yaml-cpp-0.7.0-h27087fc_2
# Linking libffi-3.4.2-h7f98852_5
# Linking bzip2-1.0.8-h7f98852_4
# Linking ncurses-6.3-h27087fc_1
# Linking fmt-9.1.0-h924138e_0
# Linking openssl-3.1.0-h0b41bf4_0
# Linking xz-5.2.6-h166bdaf_0
# Linking libzlib-1.2.13-h166bdaf_4
# Linking libnsl-2.0.0-h7f98852_0
# Linking reproc-cpp-14.2.4-hcb278e6_0
# Linking libedit-3.1.20191231-he28a2e2_2
# Linking readline-8.2-h8228510_1
# Linking libxml2-2.10.3-hfdac1af_6
# Linking zstd-1.5.2-h3eb15da_6
# Linking libssh2-1.10.0-hf14f497_3
# Linking libnghttp2-1.52.0-h61bc06f_0
# Linking libsolv-0.7.23-h3eb15da_0
# Linking tk-8.6.12-h27826a3_0
# Linking libsqlite-3.40.0-h753d276_0
# Linking krb5-1.20.1-h81ceb04_0
# Linking libarchive-3.6.2-h3d51595_0
# Linking libcurl-7.88.1-hdc1c0ab_1
# Linking libmamba-1.4.1-hcea66bb_0
# Linking tzdata-2023c-h71feb2d_0
# Linking pybind11-abi-4-hd8ed1ab_3
# Linking python-3.10.10-he550d4f_0_cpython
# Linking wheel-0.40.0-pyhd8ed1ab_0
# Linking setuptools-67.6.1-pyhd8ed1ab_0
# Linking pip-23.0.1-pyhd8ed1ab_0
# Linking pysocks-1.7.1-pyha2e5f31_6
# Linking pycparser-2.21-pyhd8ed1ab_0
# Linking charset-normalizer-2.1.1-pyhd8ed1ab_0
# Linking idna-3.4-pyhd8ed1ab_0
# Linking certifi-2022.12.7-pyhd8ed1ab_0
# Linking colorama-0.4.6-pyhd8ed1ab_0
# Linking toolz-0.12.0-pyhd8ed1ab_0
# Linking pluggy-1.0.0-pyhd8ed1ab_5
# Linking traitlets-5.9.0-pyhd8ed1ab_0
# Linking typing_extensions-4.5.0-pyha770c72_0
# Linking packaging-23.0-pyhd8ed1ab_0
# Linking boltons-23.0.0-pyhd8ed1ab_0
# Linking tqdm-4.65.0-pyhd8ed1ab_1
# Linking typing-extensions-4.5.0-hd8ed1ab_0
# Linking platformdirs-3.2.0-pyhd8ed1ab_0
# Linking ruamel.yaml.clib-0.2.7-py310h1fa729e_1
# Linking pycosat-0.6.4-py310h5764c6d_1
# Linking libmambapy-1.4.1-py310h1428755_0
# Linking cffi-1.15.1-py310h255011f_3
# Linking jupyter_core-5.3.0-py310hff52083_0
# Linking ruamel.yaml-0.17.21-py310h1fa729e_3
# Linking zstandard-0.19.0-py310hdeb6495_1
# Linking brotlipy-0.7.0-py310h5764c6d_1005
# Linking cryptography-40.0.1-py310h34c0648_0
# Linking conda-package-streaming-0.7.0-pyhd8ed1ab_1
# Linking pyopenssl-23.1.1-pyhd8ed1ab_0
# Linking conda-package-handling-2.0.2-pyh38be061_0
# Linking urllib3-1.26.15-pyhd8ed1ab_0
# Linking requests-2.28.2-pyhd8ed1ab_0
# Linking conda-23.3.0-py310hff52083_0
# Linking mamba-1.4.1-py310h51d5547_0
# Transaction finished
# + rm micromamba
# + mamba list python
# + grep '^python '
# + tr -s ' '
# + cut -d ' ' -f 1,2
# + mamba clean --all -f -y
# Will remove 1 package cache(s).
# + fix-permissions /opt/conda
# + fix-permissions /home/jovyan

# Configure container startup
ENTRYPOINT ["tini", "-g", "--"]
# Step 23/27 : ENTRYPOINT ["tini", "-g", "--"]

CMD ["start.sh"]
# Step 24/27 : CMD ["start.sh"]

# Copy local files as late as possible to avoid cache busting
COPY start.sh /usr/local/bin/
# Step 25/27 : COPY start.sh /usr/local/bin/

# Switch back to jovyan to avoid accidental container runs as root
USER ${NB_UID}
# Step 26/27 : USER 1000

WORKDIR "${HOME}"
# Step 27/27 : WORKDIR "/home/jovyan"


######################################################

# $ docker run -ti --rm jupyter/docker-stacks-foundation:c77245e2acbc bash
# (base) jovyan@cd8b00308557:~$ pwd
# /home/jovyan
# (base) jovyan@cd8b00308557:~$ ls -al
# total 28
# drwsrws--- 1 jovyan users 4096 Apr  3 03:03 .
# drwxr-xr-x 1 root   root  4096 Apr  3 03:03 ..
# -rw-rw-r-- 1 jovyan users  220 Jan  6  2022 .bash_logout
# -rw-rw-r-- 1 jovyan users 3823 Apr  3 03:03 .bashrc
# -rw-rw-r-- 1 jovyan users  807 Jan  6  2022 .profile
# -rw-rw-r-- 1 jovyan users  171 Apr  3 03:03 .wget-hsts
# drwsrwsr-x 2 jovyan users 4096 Apr  3 03:03 work
# (base) jovyan@cd8b00308557:~$
# (base) jovyan@cd8b00308557:~$ ls -al work/
# total 8
# drwsrwsr-x 2 jovyan users 4096 Apr  3 03:03 .
# drwsrws--- 1 jovyan users 4096 Apr  3 03:03 ..
# (base) jovyan@cd8b00308557:~$

# (base) jovyan@cd8b00308557:~$ ls -al /opt/conda/
# total 80
# drwsrwsr-x  1 jovyan users  4096 Apr  3 03:04 .
# drwxr-xr-x  1 root   root   4096 Apr  3 03:03 ..
# drwsrwsr-x  2 jovyan users  4096 Apr  3 03:04 bin
# drwsrwsr-x  2 jovyan users  4096 Apr  3 03:04 compiler_compat
# drwsrwsr-x  2 jovyan users  4096 Apr  3 03:04 condabin
# drwsrwsr-x  2 jovyan users  4096 Apr  3 03:04 conda-meta
# -rw-rw-r--  1 jovyan users   163 Apr  3 03:02 .condarc
# drwsrwsr-x  5 jovyan users  4096 Apr  3 03:04 etc
# drwsrwsr-x 26 jovyan users  4096 Apr  3 03:04 include
# drwsrwsr-x 17 jovyan users 12288 Apr  3 03:04 lib
# drwsrwsr-x  3 jovyan users  4096 Apr  3 03:04 libexec
# drwsrwsr-x  3 jovyan users  4096 Apr  3 03:04 man
# drwsrwsr-x  2 jovyan users  4096 Apr  3 03:04 sbin
# drwsrwsr-x 17 jovyan users  4096 Apr  3 03:04 share
# drwsrwsr-x  3 jovyan users  4096 Apr  3 03:04 shell
# drwsrwsr-x  3 jovyan users  4096 Apr  3 03:04 ssl
# drwsrwsr-x  3 jovyan users  4096 Apr  3 03:04 x86_64-conda_cos6-linux-gnu
# drwsrwsr-x  3 jovyan users  4096 Apr  3 03:04 x86_64-conda-linux-gnu
# (base) jovyan@cd8b00308557:~$
# (base) jovyan@cd8b00308557:~$ echo ${PATH}
# /opt/conda/bin:/opt/conda/condabin:/opt/conda/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
# (base) jovyan@cd8b00308557:~$

# /opt/conda/bin:
# /opt/conda/condabin:
# /opt/conda/bin:
# /usr/local/sbin:
# /usr/local/bin:
# /usr/sbin:
# /usr/bin:
# /sbin:
# /bin

# (base) jovyan@cd8b00308557:~$ ls -al /opt/conda/bin
# total 28608
# drwsrwsr-x 2 jovyan users     4096 Apr  3 03:04 .
# drwsrwsr-x 1 jovyan users     4096 Apr  3 03:04 ..
# lrwxrwxrwx 1 jovyan users        9 Apr  3 03:04 2to3 -> 2to3-3.10
# -rwxrwxr-x 1 jovyan users      102 Apr  3 03:04 2to3-3.10
# -rwxrwxr-x 1 jovyan users    34408 Oct 27  2021 acountry
# -rwxrwxr-x 1 jovyan users      184 Apr  3 03:04 activate
# -rwxrwxr-x 1 jovyan users    30672 Oct 27  2021 adig
# -rwxrwxr-x 1 jovyan users    21744 Oct 27  2021 ahost
# -rwxrwxr-x 1 jovyan users    97048 Dec 12 18:50 bsdcat
# -rwxrwxr-x 1 jovyan users   831544 Dec 12 18:50 bsdcpio
# -rwxrwxr-x 1 jovyan users   885848 Dec 12 18:50 bsdtar
# -rwxrwxr-x 1 jovyan users   384272 Nov 28  2020 bunzip2
# -rwxrwxr-x 1 jovyan users   384272 Nov 28  2020 bzcat
# lrwxrwxrwx 1 jovyan users        6 Apr  3 03:04 bzcmp -> bzdiff
# -rwxrwxr-x 1 jovyan users     2140 Nov 28  2020 bzdiff
# lrwxrwxrwx 1 jovyan users        6 Apr  3 03:04 bzegrep -> bzgrep
# lrwxrwxrwx 1 jovyan users        6 Apr  3 03:04 bzfgrep -> bzgrep
# -rwxrwxr-x 1 jovyan users     2054 Nov 28  2020 bzgrep
# -rwxrwxr-x 1 jovyan users   384272 Nov 28  2020 bzip2
# -rwxrwxr-x 1 jovyan users    35944 Nov 28  2020 bzip2recover
# lrwxrwxrwx 1 jovyan users        6 Apr  3 03:04 bzless -> bzmore
# -rwxrwxr-x 1 jovyan users     1259 Nov 28  2020 bzmore
# lrwxrwxrwx 1 jovyan users        3 Apr  3 03:04 captoinfo -> tic
# -rwxrwxr-x 1 jovyan users    14320 Apr  7  2022 clear
# -rwxrwxr-x 1 jovyan users      824 Apr  3 03:04 compile_et
# -rwxrwxr-x 1 jovyan users      511 Apr  3 03:04 conda
# -rwxrwxr-x 1 jovyan users    21664 Dec 14 22:05 conda2solv
# -rwxrwxr-x 1 jovyan users      148 Apr  3 03:04 conda-env
# -rwxrwxr-x 1 jovyan users      237 Apr  3 03:04 cph
# -rwxrwxr-x 1 jovyan users     6858 Apr  3 03:04 c_rehash
# -rwxrwxr-x 1 jovyan users     7386 Apr  3 03:04 curl-config
# -rwxrwxr-x 1 jovyan users      272 Apr  3 03:04 deactivate
# -rwxrwxr-x 1 jovyan users    34904 Mar 20 12:12 derb
# -rwxrwxr-x 1 jovyan users    26336 Dec 14 22:05 dumpsolv
# -rwxrwxr-x 1 jovyan users    22152 Mar 20 12:12 genbrk
# -rwxrwxr-x 1 jovyan users    17448 Mar 20 12:12 gencfu
# -rwxrwxr-x 1 jovyan users    35136 Mar 20 12:12 gencnval
# -rwxrwxr-x 1 jovyan users    37496 Mar 20 12:12 gendict
# -rwxrwxr-x 1 jovyan users   203336 Mar 20 12:12 genrb
# -rwxrwxr-x 1 jovyan users    31928 Dec 15 08:09 gss-client
# -rwxrwxr-x 1 jovyan users    44640 Apr  3 03:04 iconv
# -rwxrwxr-x 1 jovyan users    22778 Apr  3 03:04 icu-config
# -rwxrwxr-x 1 jovyan users    70504 Mar 20 12:12 icuexportdata
# -rwxrwxr-x 1 jovyan users    17072 Mar 20 12:12 icuinfo
# lrwxrwxrwx 1 jovyan users        8 Apr  3 03:04 idle3 -> idle3.10
# -rwxrwxr-x 1 jovyan users      100 Apr  3 03:04 idle3.10
# -rwxrwxr-x 1 jovyan users    63544 Apr  7  2022 infocmp
# lrwxrwxrwx 1 jovyan users        3 Apr  3 03:04 infotocap -> tic
# -rwxrwxr-x 1 jovyan users    26072 Dec 14 22:05 installcheck
# -rwxrwxr-x 1 jovyan users     1008 Apr  3 03:04 jsondiff
# -rwxrwxr-x 1 jovyan users     3863 Apr  3 03:04 jsonpatch
# -rw-rw-r-- 1 jovyan users     1842 Apr  3 03:04 jsonpointer
# -rwxrwxr-x 1 jovyan users      228 Apr  3 03:04 jupyter
# -rwxrwxr-x 1 jovyan users      228 Apr  3 03:04 jupyter-migrate
# -rwxrwxr-x 1 jovyan users      233 Apr  3 03:04 jupyter-troubleshoot
# -rwxrwxr-x 1 jovyan users     2004 Dec 15 08:09 k5srvutil
# -rwxrwxr-x 1 jovyan users   102960 Dec 15 08:09 kadmin
# -rwxrwxr-x 1 jovyan users    17120 Dec 15 08:09 kdestroy
# -rwxrwxr-x 1 jovyan users    76064 Mar  1  2022 keyctl
# -rwxrwxr-x 1 jovyan users    41376 Dec 15 08:09 kinit
# -rwxrwxr-x 1 jovyan users    36472 Dec 15 08:09 klist
# -rwxrwxr-x 1 jovyan users    21528 Dec 15 08:09 kpasswd
# -rwxrwxr-x 1 jovyan users     7028 Apr  3 03:04 krb5-config
# -rw-rw-r-- 1 jovyan users    61176 Dec 15 08:09 ksu
# -rwxrwxr-x 1 jovyan users    16776 Dec 15 08:09 kswitch
# -rwxrwxr-x 1 jovyan users    42648 Dec 15 08:09 ktutil
# -rwxrwxr-x 1 jovyan users    27472 Dec 15 08:09 kvno
# -rwxrwxr-x 1 jovyan users   211008 Jan 26 09:57 lz4
# lrwxrwxrwx 1 jovyan users        3 Apr  3 03:04 lz4c -> lz4
# lrwxrwxrwx 1 jovyan users        3 Apr  3 03:04 lz4cat -> lz4
# lrwxrwxrwx 1 jovyan users        2 Apr  3 03:04 lzcat -> xz
# lrwxrwxrwx 1 jovyan users        6 Apr  3 03:04 lzcmp -> xzdiff
# lrwxrwxrwx 1 jovyan users        6 Apr  3 03:04 lzdiff -> xzdiff
# lrwxrwxrwx 1 jovyan users        6 Apr  3 03:04 lzegrep -> xzgrep
# lrwxrwxrwx 1 jovyan users        6 Apr  3 03:04 lzfgrep -> xzgrep
# lrwxrwxrwx 1 jovyan users        6 Apr  3 03:04 lzgrep -> xzgrep
# lrwxrwxrwx 1 jovyan users        6 Apr  3 03:04 lzless -> xzless
# lrwxrwxrwx 1 jovyan users        2 Apr  3 03:04 lzma -> xz
# -rwxrwxr-x 1 jovyan users    17368 Aug 12  2022 lzmadec
# -rwxrwxr-x 1 jovyan users    17280 Apr  3 03:04 lzmainfo
# lrwxrwxrwx 1 jovyan users        6 Apr  3 03:04 lzmore -> xzmore
# -rwxrwxr-x 1 jovyan users    67696 Mar 20 12:12 makeconv
# -rwxrwxr-x 1 jovyan users      219 Apr  3 03:04 mamba
# -rwxrwxr-x 1 jovyan users   659976 Apr  3 03:04 mamba-package
# -rwxrwxr-x 1 jovyan users    21840 Dec 14 22:05 mergesolv
# -rwxrwxr-x 1 jovyan users     8793 Apr  3 03:04 ncurses6-config
# -rwxrwxr-x 1 jovyan users     8797 Apr  3 03:04 ncursesw6-config
# -rwxrwxr-x 1 jovyan users   258360 Mar  1 13:41 nghttp
# -rwxrwxr-x 1 jovyan users   188712 Mar  1 13:41 nghttpd
# -rwxrwxr-x 1 jovyan users  1305960 Apr  3 03:04 nghttpx
# -rwxrwxr-x 1 jovyan users      256 Apr  3 03:04 normalizer
# -rwxrwxr-x 1 jovyan users  1102992 Mar 14 19:36 openssl
# -rwxrwxr-x 1 jovyan users      233 Apr  3 03:04 pip
# -rwxrwxr-x 1 jovyan users      233 Apr  3 03:04 pip3
# -rwxrwxr-x 1 jovyan users    55000 Mar 20 12:12 pkgdata
# lrwxrwxrwx 1 jovyan users        9 Apr  3 03:04 pydoc -> pydoc3.10
# lrwxrwxrwx 1 jovyan users        9 Apr  3 03:04 pydoc3 -> pydoc3.10
# -rwxrwxr-x 1 jovyan users       85 Apr  3 03:04 pydoc3.10
# lrwxrwxrwx 1 jovyan users       10 Apr  3 03:04 python -> python3.10
# lrwxrwxrwx 1 jovyan users       10 Apr  3 03:04 python3 -> python3.10
# lrwxrwxrwx 1 jovyan users       10 Apr  3 03:04 python3.1 -> python3.10
# -rwxrwxr-x 1 jovyan users 17606648 Apr  3 03:04 python3.10
# -rwxrwxr-x 1 jovyan users     3493 Apr  3 03:04 python3.10-config
# lrwxrwxrwx 1 jovyan users       17 Apr  3 03:04 python3-config -> python3.10-config
# -rwxrwxr-x 1 jovyan users    22464 Dec 14 22:05 repo2solv
# lrwxrwxrwx 1 jovyan users        4 Apr  3 03:04 reset -> tset
# -rwxrwxr-x 1 jovyan users    21600 Dec 15 08:09 sclient
# -rwxrwxr-x 1 jovyan users    21496 Dec 15 08:09 sim_client
# -rwxrwxr-x 1 jovyan users    30392 Feb 16  2022 sqlite3_analyzer
# -rwxrwxr-x 1 jovyan users    22504 Apr  7  2022 tabs
# lrwxrwxrwx 1 jovyan users        8 Apr  3 03:04 tclsh -> tclsh8.6
# -rwxrwxr-x 1 jovyan users    16144 Feb 16  2022 tclsh8.6
# -rwxrwxr-x 1 jovyan users    27712 Dec 14 22:05 testsolv
# -rwxrwxr-x 1 jovyan users    88152 Apr  7  2022 tic
# -rwxrwxr-x 1 jovyan users    22504 Apr  7  2022 toe
# -rwxrwxr-x 1 jovyan users    22536 Apr  7  2022 tput
# -rwxrwxr-x 1 jovyan users      219 Apr  3 03:04 tqdm
# -rwxrwxr-x 1 jovyan users    30704 Apr  7  2022 tset
# lrwxrwxrwx 1 jovyan users        3 Apr  3 03:04 unlz4 -> lz4
# lrwxrwxrwx 1 jovyan users        2 Apr  3 03:04 unlzma -> xz
# lrwxrwxrwx 1 jovyan users        2 Apr  3 03:04 unxz -> xz
# lrwxrwxrwx 1 jovyan users        4 Apr  3 03:04 unzstd -> zstd
# -rwxrwxr-x 1 jovyan users    22048 Dec 15 08:09 uuclient
# -rwxrwxr-x 1 jovyan users      220 Apr  3 03:04 wheel
# lrwxrwxrwx 1 jovyan users        7 Apr  3 03:04 wish -> wish8.6
# -rwxrwxr-x 1 jovyan users    16296 Feb 16  2022 wish8.6
# lrwxrwxrwx 1 jovyan users       25 Apr  3 03:04 x86_64-conda_cos6-linux-gnu-ld -> x86_64-conda-linux-gnu-ld
# -rwxrwxr-x 1 jovyan users  2442288 Jan 27 15:39 x86_64-conda-linux-gnu-ld
# -rwxrwxr-x 1 jovyan users     1903 Apr  3 03:04 xml2-config
# -rwxrwxr-x 1 jovyan users    26472 Apr  3 03:04 xmlcatalog
# -rwxrwxr-x 1 jovyan users    89104 Apr  3 03:04 xmllint
# -rwxrwxr-x 1 jovyan users    95336 Apr  3 03:04 xz
# lrwxrwxrwx 1 jovyan users        2 Apr  3 03:04 xzcat -> xz
# lrwxrwxrwx 1 jovyan users        6 Apr  3 03:04 xzcmp -> xzdiff
# -rwxrwxr-x 1 jovyan users    17368 Aug 12  2022 xzdec
# -rwxrwxr-x 1 jovyan users     7307 Aug 12  2022 xzdiff
# lrwxrwxrwx 1 jovyan users        6 Apr  3 03:04 xzegrep -> xzgrep
# lrwxrwxrwx 1 jovyan users        6 Apr  3 03:04 xzfgrep -> xzgrep
# -rwxrwxr-x 1 jovyan users    10293 Aug 12  2022 xzgrep
# -rwxrwxr-x 1 jovyan users     1821 Aug 12  2022 xzless
# -rwxrwxr-x 1 jovyan users     2198 Aug 12  2022 xzmore
# -rwxrwxr-x 1 jovyan users   168624 Jan 20 19:56 zstd
# lrwxrwxrwx 1 jovyan users        4 Apr  3 03:04 zstdcat -> zstd
# -rwxrwxr-x 1 jovyan users     3869 Jan 20  2022 zstdgrep
# -rwxrwxr-x 1 jovyan users       30 Jan 20  2022 zstdless
# lrwxrwxrwx 1 jovyan users        4 Apr  3 03:04 zstdmt -> zstd
# (base) jovyan@cd8b00308557:~$
# (base) jovyan@cd8b00308557:~$ ls -al /opt/conda/condabin
# total 12
# drwsrwsr-x 2 jovyan users 4096 Apr  3 03:04 .
# drwsrwsr-x 1 jovyan users 4096 Apr  3 03:04 ..
# -rwxrwxr-x 1 jovyan users  511 Apr  3 03:04 conda
# lrwxrwxrwx 1 jovyan users   12 Apr  3 03:04 mamba -> ../bin/mamba
# (base) jovyan@cd8b00308557:~$

# jupyter
# jupyter-migrate
# jupyter-troubleshoot

# (base) jovyan@cd8b00308557:~$ jupyter -h
# usage: jupyter [-h] [--version] [--config-dir] [--data-dir] [--runtime-dir] [--paths] [--json] [--debug] [subcommand]

# Jupyter: Interactive Computing

# positional arguments:
#   subcommand     the subcommand to launch

# options:
#   -h, --help     show this help message and exit
#   --version      show the versions of core jupyter packages and exit
#   --config-dir   show Jupyter config dir
#   --data-dir     show Jupyter data dir
#   --runtime-dir  show Jupyter runtime dir
#   --paths        show all Jupyter paths. Add --json for machine-readable format.
#   --json         output paths as machine-readable json
#   --debug        output debug information about paths

# Available subcommands: migrate troubleshoot
# (base) jovyan@cd8b00308557:~$
# (base) jovyan@cd8b00308557:~$ jupyter --version
# Selected Jupyter core packages...
# IPython          : not installed
# ipykernel        : not installed
# ipywidgets       : not installed
# jupyter_client   : not installed
# jupyter_core     : 5.3.0
# jupyter_server   : not installed
# jupyterlab       : not installed
# nbclient         : not installed
# nbconvert        : not installed
# nbformat         : not installed
# notebook         : not installed
# qtconsole        : not installed
# traitlets        : 5.9.0
# (base) jovyan@cd8b00308557:~$
