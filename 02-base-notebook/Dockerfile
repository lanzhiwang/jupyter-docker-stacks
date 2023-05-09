# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.


# docker build  --rm --force-rm -t jupyter/base-notebook:latest ./base-notebook --build-arg OWNER=jupyter

ARG OWNER=jupyter
# Step 1/20 : ARG OWNER=jupyter

ARG BASE_CONTAINER=$OWNER/docker-stacks-foundation
# Step 2/20 : ARG BASE_CONTAINER=jupyter/docker-stacks-foundation

FROM $BASE_CONTAINER
# Step 3/20 : FROM jupyter/docker-stacks-foundation

LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"
# Step 4/20 : LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"

# Fix: https://github.com/hadolint/hadolint/wiki/DL4006
# Fix: https://github.com/koalaman/shellcheck/wiki/SC3014
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
# Step 5/20 : SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root
# Step 6/20 : USER root

# Install all OS dependencies for notebook server that starts but lacks all
# features (e.g., download as all possible file formats)
RUN apt-get update --yes && \
    apt-get install --yes --no-install-recommends \
    fonts-liberation \
    # - pandoc is used to convert notebooks to html files
    #   it's not present in aarch64 ubuntu image, so we install it here
    pandoc \
    # - run-one - a wrapper script that runs no more
    #   than one unique  instance  of  some  command with a unique set of arguments,
    #   we use `run-one-constantly` to support `RESTARTABLE` option
    run-one && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
# Step 7/20 : RUN
# apt-get update --yes &&
# apt-get install --yes --no-install-recommends fonts-liberation pandoc run-one &&
# apt-get clean &&
# rm -rf /var/lib/apt/lists/*

USER ${NB_UID}
# Step 8/20 : USER 1000

# Install Jupyter Notebook, Lab, and Hub
# Generate a notebook server config
# Cleanup temporary files
# Correct permissions
# Do all this in a single RUN command to avoid duplicating all of the
# files across image layers when the permissions change
WORKDIR /tmp
# Step 9/20 : WORKDIR /tmp

RUN mamba install --yes \
    'notebook' \
    'jupyterhub' \
    'jupyterlab' && \
    jupyter notebook --generate-config && \
    mamba clean --all -f -y && \
    npm cache clean --force && \
    jupyter lab clean && \
    rm -rf "/home/${NB_USER}/.cache/yarn" && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"
# Step 10/20 : RUN
# mamba install --yes 'notebook' 'jupyterhub' 'jupyterlab' &&
# jupyter notebook --generate-config &&
# mamba clean --all -f -y &&
# npm cache clean --force &&
# jupyter lab clean &&
# rm -rf "/home/jovyan/.cache/yarn" &&
# fix-permissions "/opt/conda" &&
# fix-permissions "/home/jovyan"

ENV JUPYTER_PORT=8888
# Step 11/20 : ENV JUPYTER_PORT=8888

EXPOSE $JUPYTER_PORT
# Step 12/20 : EXPOSE $JUPYTER_PORT

# Configure container startup
CMD ["start-notebook.sh"]
# Step 13/20 : CMD ["start-notebook.sh"]

# Copy local files as late as possible to avoid cache busting
COPY start-notebook.sh start-singleuser.sh /usr/local/bin/
# Step 14/20 : COPY start-notebook.sh start-singleuser.sh /usr/local/bin/

# Currently need to have both jupyter_notebook_config and jupyter_server_config to support classic and lab
COPY jupyter_server_config.py docker_healthcheck.py /etc/jupyter/
# Step 15/20 : COPY jupyter_server_config.py docker_healthcheck.py /etc/jupyter/

# Fix permissions on /etc/jupyter as root
USER root
# Step 16/20 : USER root

# Legacy for Jupyter Notebook Server, see: [#1205](https://github.com/jupyter/docker-stacks/issues/1205)
RUN sed -re "s/c.ServerApp/c.NotebookApp/g" \
    /etc/jupyter/jupyter_server_config.py > /etc/jupyter/jupyter_notebook_config.py && \
    fix-permissions /etc/jupyter/
# Step 17/20 : RUN
# sed -re "s/c.ServerApp/c.NotebookApp/g" /etc/jupyter/jupyter_server_config.py > /etc/jupyter/jupyter_notebook_config.py &&
# fix-permissions /etc/jupyter/

# HEALTHCHECK documentation: https://docs.docker.com/engine/reference/builder/#healthcheck
# This healtcheck works well for `lab`, `notebook`, `nbclassic`, `server` and `retro` jupyter commands
# https://github.com/jupyter/docker-stacks/issues/915#issuecomment-1068528799
HEALTHCHECK --interval=5s --timeout=3s --start-period=5s --retries=3 \
    CMD /etc/jupyter/docker_healthcheck.py || exit 1
# Step 18/20 : HEALTHCHECK --interval=5s --timeout=3s --start-period=5s --retries=3 CMD /etc/jupyter/docker_healthcheck.py || exit 1

# Switch back to jovyan to avoid accidental container runs as root
USER ${NB_UID}
# Step 19/20 : USER 1000

WORKDIR "${HOME}"
# Step 20/20 : WORKDIR "/home/jovyan"


######################################################

# $ docker run -ti --rm jupyter/base-notebook:c77245e2acbc bash
# (base) jovyan@a94fd88c7d56:~$ jupyter -h
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

# Available subcommands: bundlerextension dejavu events execute fileid kernel kernelspec lab labextension labhub migrate nbclassic nbconvert nbextension notebook run server serverextension troubleshoot trust

# bundlerextension
# dejavu
# events
# execute
# fileid
# kernel
# kernelspec
# lab
# labextension
# labhub
# migrate
# nbclassic
# nbconvert
# nbextension
# notebook
# run
# server
# serverextension
# troubleshoot
# trust

# (base) jovyan@a94fd88c7d56:~$ ll /opt/conda/bin/ | grep jupyter
# -rwxrwxr-x 1 jovyan users       228 Apr  3 03:04 jupyter*
# -rwxrwxr-x 1 jovyan users       245 Apr  3 03:08 jupyter-bundlerextension*
# -rwxrwxr-x 1 jovyan users       247 Apr  3 03:08 jupyter-dejavu*
# -rwxrwxr-x 1 jovyan users       255 Apr  3 03:08 jupyter-events*
# -rwxrwxr-x 1 jovyan users       249 Apr  3 03:08 jupyter-execute*
# -rwxrwxr-x 1 jovyan users       236 Apr  3 03:08 jupyter-fileid*
# -rwxrwxr-x 1 jovyan users       225 Apr  3 03:08 jupyterhub*
# -rwxrwxr-x 1 jovyan users       232 Apr  3 03:08 jupyterhub-singleuser*
# -rwxrwxr-x 1 jovyan users       235 Apr  3 03:08 jupyter-kernel*
# -rwxrwxr-x 1 jovyan users       273 Apr  3 03:08 jupyter-kernelspec*
# -rwxrwxr-x 1 jovyan users       228 Apr  3 03:08 jupyter-lab*
# -rwxrwxr-x 1 jovyan users       235 Apr  3 03:08 jupyter-labextension*
# -rwxrwxr-x 1 jovyan users       231 Apr  3 03:08 jupyter-labhub*
# -rwxrwxr-x 1 jovyan users       228 Apr  3 03:04 jupyter-migrate*
# -rwxrwxr-x 1 jovyan users       232 Apr  3 03:08 jupyter-nbclassic*
# -rwxrwxr-x 1 jovyan users       246 Apr  3 03:08 jupyter-nbclassic-bundlerextension*
# -rwxrwxr-x 1 jovyan users       233 Apr  3 03:08 jupyter-nbclassic-extension*
# -rwxrwxr-x 1 jovyan users       237 Apr  3 03:08 jupyter-nbclassic-serverextension*
# -rwxrwxr-x 1 jovyan users       233 Apr  3 03:08 jupyter-nbconvert*
# -rwxrwxr-x 1 jovyan users       232 Apr  3 03:08 jupyter-nbextension*
# -rwxrwxr-x 1 jovyan users       231 Apr  3 03:08 jupyter-notebook*
# -rwxrwxr-x 1 jovyan users       232 Apr  3 03:08 jupyter-run*
# -rwxrwxr-x 1 jovyan users       235 Apr  3 03:08 jupyter-server*
# -rwxrwxr-x 1 jovyan users       236 Apr  3 03:08 jupyter-serverextension*
# -rwxrwxr-x 1 jovyan users       233 Apr  3 03:04 jupyter-troubleshoot*
# -rwxrwxr-x 1 jovyan users       264 Apr  3 03:08 jupyter-trust*
# (base) jovyan@a94fd88c7d56:~$

# (base) jovyan@a94fd88c7d56:~$ jupyter --version
# Selected Jupyter core packages...
# IPython          : 8.12.0
# ipykernel        : 6.22.0
# ipywidgets       : not installed
# jupyter_client   : 8.1.0
# jupyter_core     : 5.3.0
# jupyter_server   : 2.5.0
# jupyterlab       : 3.6.3
# nbclient         : 0.7.2
# nbconvert        : 7.2.10
# nbformat         : 5.8.0
# notebook         : 6.5.3
# qtconsole        : not installed
# traitlets        : 5.9.0
# (base) jovyan@a94fd88c7d56:~$
