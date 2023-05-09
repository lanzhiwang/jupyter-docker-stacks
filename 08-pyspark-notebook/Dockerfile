# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.


# docker build  --rm --force-rm -t jupyter/pyspark-notebook:latest ./pyspark-notebook --build-arg OWNER=jupyter

ARG OWNER=jupyter
# Step 1/24 : ARG OWNER=jupyter

ARG BASE_CONTAINER=$OWNER/scipy-notebook
# Step 2/24 : ARG BASE_CONTAINER=jupyter/scipy-notebook

FROM $BASE_CONTAINER
# Step 3/24 : FROM jupyter/scipy-notebook

LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"
# Step 4/24 : LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"

# Fix: https://github.com/hadolint/hadolint/wiki/DL4006
# Fix: https://github.com/koalaman/shellcheck/wiki/SC3014
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
# Step 5/24 : SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root
# Step 6/24 : USER root

# Spark dependencies
# Default values can be overridden at build time
# (ARGS are in lower case to distinguish them from ENV)
ARG spark_version="3.3.2"
# Step 7/24 : ARG spark_version="3.3.2"

ARG hadoop_version="3"
# Step 8/24 : ARG hadoop_version="3"

ARG scala_version
# Step 9/24 : ARG scala_version

ARG spark_checksum="4cd2396069fbe0f8efde2af4fd301bf46f8c6317e9dea1dd42a405de6a38380635d49b17972cb92c619431acece2c3af4c23bfdf193cedb3ea913ed69ded23a1"
# Step 10/24 : ARG spark_checksum="4cd2396069fbe0f8efde2af4fd301bf46f8c6317e9dea1dd42a405de6a38380635d49b17972cb92c619431acece2c3af4c23bfdf193cedb3ea913ed69ded23a1"

ARG openjdk_version="17"
# Step 11/24 : ARG openjdk_version="17"

ENV APACHE_SPARK_VERSION="${spark_version}" \
    HADOOP_VERSION="${hadoop_version}"
# Step 12/24 : ENV
# APACHE_SPARK_VERSION="3.3.2"
# HADOOP_VERSION="3"

RUN apt-get update --yes && \
    apt-get install --yes --no-install-recommends \
    "openjdk-${openjdk_version}-jre-headless" \
    ca-certificates-java && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
# Step 13/24 : RUN
# apt-get update --yes &&
# apt-get install --yes --no-install-recommends "openjdk-17-jre-headless" ca-certificates-java &&
# apt-get clean &&
# rm -rf /var/lib/apt/lists/*

# Spark installation
WORKDIR /tmp
# Step 14/24 : WORKDIR /tmp

# You need to use https://archive.apache.org/dist/ website if you want to download old Spark versions
# But it seems to be slower, that's why we use recommended site for download
RUN if [ -z "${scala_version}" ]; then \
    wget -qO "spark.tgz" "https://dlcdn.apache.org/spark/spark-${APACHE_SPARK_VERSION}/spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz"; \
  else \
    wget -qO "spark.tgz" "https://dlcdn.apache.org/spark/spark-${APACHE_SPARK_VERSION}/spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}-scala${scala_version}.tgz"; \
  fi && \
  echo "${spark_checksum} *spark.tgz" | sha512sum -c - && \
  tar xzf "spark.tgz" -C /usr/local --owner root --group root --no-same-owner && \
  rm "spark.tgz"
# Step 15/24 : RUN
# if [ -z "${scala_version}" ]; then
#     wget -qO "spark.tgz" "https://dlcdn.apache.org/spark/spark-3.3.2/spark-3.3.2-bin-hadoop3.tgz";
# else
#     wget -qO "spark.tgz" "https://dlcdn.apache.org/spark/spark-3.3.2/spark-3.3.2-bin-hadoop3-scala${scala_version}.tgz";
# fi &&
# echo "${spark_checksum} *spark.tgz" | sha512sum -c - &&
# tar xzf "spark.tgz" -C /usr/local --owner root --group root --no-same-owner &&
# rm "spark.tgz"

# Configure Spark
ENV SPARK_HOME=/usr/local/spark
# Step 16/24 : ENV SPARK_HOME=/usr/local/spark

ENV SPARK_OPTS="--driver-java-options=-Xms1024M --driver-java-options=-Xmx4096M --driver-java-options=-Dlog4j.logLevel=info" \
    PATH="${PATH}:${SPARK_HOME}/bin"
# Step 17/24 : ENV
# SPARK_OPTS="--driver-java-options=-Xms1024M --driver-java-options=-Xmx4096M --driver-java-options=-Dlog4j.logLevel=info"
# PATH="${PATH}:/usr/local/spark/bin"

RUN if [ -z "${scala_version}" ]; then \
    ln -s "spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}" "${SPARK_HOME}"; \
  else \
    ln -s "spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}-scala${scala_version}" "${SPARK_HOME}"; \
  fi && \
  # Add a link in the before_notebook hook in order to source automatically PYTHONPATH && \
  mkdir -p /usr/local/bin/before-notebook.d && \
  ln -s "${SPARK_HOME}/sbin/spark-config.sh" /usr/local/bin/before-notebook.d/spark-config.sh
# Step 18/24 : RUN
# if [ -z "${scala_version}" ]; then
#     ln -s "spark-3.3.2-bin-hadoop3" "/usr/local/spark";
# else
#     ln -s "spark-3.3.2-bin-hadoop3-scala${scala_version}" "/usr/local/spark";
# fi &&
# mkdir -p /usr/local/bin/before-notebook.d &&
# ln -s "/usr/local/spark/sbin/spark-config.sh" /usr/local/bin/before-notebook.d/spark-config.sh

# Configure IPython system-wide
COPY ipython_kernel_config.py "/etc/ipython/"
# Step 19/24 : COPY ipython_kernel_config.py "/etc/ipython/"

RUN fix-permissions "/etc/ipython/"
# Step 20/24 : RUN fix-permissions "/etc/ipython/"

USER ${NB_UID}
# Step 21/24 : USER 1000

# Install pyarrow
RUN mamba install --yes \
    'pyarrow' && \
    mamba clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"
# Step 22/24 : RUN
# mamba install --yes 'pyarrow' &&
# mamba clean --all -f -y &&
# fix-permissions "/opt/conda" &&
# fix-permissions "/home/jovyan"

WORKDIR "${HOME}"
# Step 23/24 : WORKDIR /home/jovyan

EXPOSE 4040
# Step 24/24 : 4040
