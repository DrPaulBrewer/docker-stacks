# Copyright (c) Jupyter Development Team
# Copyright (c) Paul Brewer Economic and Financial Technology Consulting LLC, changes vs. all-spark-notebook
# Distributed under the terms of the Modified BSD License.

FROM drpaulbrewer/minimal-notebook-ubuntu

MAINTAINER Paul Brewer <drpaulbrewer@eaftc.com>

USER root

# Util to help with kernel spec later
RUN apt-get -y update && apt-get -y install jq

# Spark dependencies -- use Paul's custom build of spark maintainence snapshot on openjdk8
ENV APACHE_SPARK_VERSION spark-1.6.2-SNAPSHOT-bin-custom-spark-1.6
ADD spark-1.6.2-SNAPSHOT-bin-custom-spark-1.6.tgz /usr/local/
RUN apt-get -y update && \
    apt-get install -y --no-install-recommends openjdk-8-jdk-headless && \
    apt-get clean
RUN cd /usr/local && ln -s spark-1.6.2-SNAPSHOT-bin-custom-spark-1.6 spark

ENV APACHE_SPARK_VERSION 1.6.0
ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64

# toree -- get the latest release from the official pip site instead of an old, frozen github hex commit
# torees default kernel.json forces some obnoxious settings, erasing our SPARK_OPTS. "An eye for an eye" works here.
COPY usr-local-share-jupyter-kernels-toree-kernel.json /tmp/
RUN pip install toree \
  && jupyter toree install \
  && cp /tmp/usr-local-share-jupyter-kernels-toree-kernel.json /usr/local/share/jupyter/kernels/toree/kernel.json


# Mesos dependencies (Paul -- unchanged)
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv E56151BF && \
    DISTRO=debian && \
    CODENAME=wheezy && \
    echo "deb http://repos.mesosphere.io/${DISTRO} ${CODENAME} main" > /etc/apt/sources.list.d/mesosphere.list && \
    apt-get -y update && \
    apt-get --no-install-recommends -y --force-yes install mesos=0.22.1-1.0.debian78 && \
    apt-get clean


# Spark and Mesos pointers
ENV SPARK_HOME /usr/local/spark
ENV R_LIBS_USER $SPARK_HOME/R/lib
ENV PYTHONPATH $SPARK_HOME/python:$SPARK_HOME/python/lib/py4j-0.9-src.zip
ENV MESOS_NATIVE_LIBRARY /usr/local/lib/libmesos.so

# Target driver memory param for large google/amazon compute VM
# may be changed at run time with docker -e SPARK_OPTS="new opts" ...
# -- Paul
ENV SPARK_OPTS --driver-memory 24G --driver-java-options=-Dlog4j.logLevel=info

# R pre-requisites
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    fonts-dejavu \
    gfortran \
    gcc && apt-get clean

RUN chown -R jovyan:users /home/jovyan

USER jovyan

# Install Python 3 packages
RUN conda install --yes \
    'ipywidgets=4.1*' \
    'pandas=0.17*' \
    'matplotlib=1.5*' \
    'scipy=0.17*' \
    'seaborn=0.7*' \
    'scikit-learn=0.17*' \
    && conda clean -yt

# Install Python 2 packages and kernel spec
RUN conda create --yes -p $CONDA_DIR/envs/python2 python=2.7 \
    'ipython=4.1*' \
    'ipywidgets=4.1*' \
    'pandas=0.17*' \
    'matplotlib=1.5*' \
    'scipy=0.17*' \
    'seaborn=0.7*' \
    'scikit-learn=0.17*' \
    pyzmq \
    && conda clean -yt

# R packages
RUN conda config --add channels r
RUN conda install --yes \
    'r-base=3.2*' \
    'r-irkernel=0.5*' \
    'r-ggplot2=1.0*' \
    'r-rcurl=1.95*' && conda clean -yt

# Install Python 2 kernel spec into the Python 3 conda environment which
# runs the notebook server
RUN bash -c '. activate python2 && \
    python -m ipykernel.kernelspec --prefix=$CONDA_DIR && \
    . deactivate'
# Set PYSPARK_HOME in the python2 spec

RUN jq --arg v "$CONDA_DIR/envs/python2/bin/python" \
        '.["env"]["PYSPARK_PYTHON"]=$v' \
        $CONDA_DIR/share/jupyter/kernels/python2/kernel.json > /tmp/kernel.json && \
        mv /tmp/kernel.json $CONDA_DIR/share/jupyter/kernels/python2/kernel.json

