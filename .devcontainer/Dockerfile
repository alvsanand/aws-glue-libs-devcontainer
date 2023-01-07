FROM mcr.microsoft.com/vscode/devcontainers/base:ubuntu-22.04

SHELL ["/bin/bash", "-c"]

USER root

ARG USERNAME=vscode

RUN apt-get update \
    && apt-get install -y curl wget unzip zip vim jq software-properties-common

# Install Python
ARG PYTHON_VERSION="3.10"

RUN add-apt-repository -y ppa:deadsnakes/ppa && \
    apt-get update && \
    apt install -y python$PYTHON_VERSION python3-pip

# Install sudo
RUN apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

USER $USERNAME
WORKDIR "/home/${USERNAME}"
ENV HOME="/home/${USERNAME}"

# Install Java & related
RUN curl -s "https://get.sdkman.io" | bash

ARG JDK_VERSION="11"
ARG SCALA_VERSION="2.12"
ARG MAVEN_VERSION="3.8.7"

RUN source "$HOME/.sdkman/bin/sdkman-init.sh" \
    && FINAL_JDK_VERSION=$(sdk list java | sed -En 's/.+ ('${JDK_VERSION}'(.+)-amzn).*/\1/p') \
    && sdk install java "$FINAL_JDK_VERSION" \
    && FINAL_SCALA_VERSION=$(sdk list scala | sed -En 's/.+ ('${SCALA_VERSION}'(.+)-amzn).*/\1/p') \
    && sdk install scala "$FINAL_SCALA_VERSION" \
    && sdk install maven "$MAVEN_VERSION"

# Install aws-glue-libs
ARG AWS_GLUE_SPARK_URL="https://aws-glue-etl-artifacts.s3.amazonaws.com/glue-4.0/spark-3.3.0-amzn-1-bin-3.3.3-amzn-0.tgz"
ARG AWS_GLUE_VERSION="4.0"

RUN wget -qO- "$AWS_GLUE_SPARK_URL" | tar zxf - 2>/dev/null \
    && mv build/spark spark && rm -Rf build

RUN wget -qO- "https://github.com/awslabs/aws-glue-libs/archive/refs/tags/v$AWS_GLUE_VERSION.tar.gz" | tar zxf - 2>/dev/null \
    && mv aws-glue-libs-* aws-glue-libs \
    && python3 -m pip install boto3 pytest

RUN source "$HOME/.sdkman/bin/sdkman-init.sh" \
    && cd aws-glue-libs \
    && mvn clean package dependency:copy-dependencies

# Configure profile
RUN echo 'alias python=python3' >> "$HOME/.bash_profile"
RUN echo 'export PATH="${HOME}/aws-glue-libs/bin:${HOME}/spark/bin:${PATH}":${HOME}/.local/bin' >> "$HOME/.bash_profile"
RUN echo 'export SPARK_HOME="${HOME}/spark"' >> "$HOME/.bash_profile"
RUN echo 'source "$HOME/.bash_profile"' >> "$HOME/.bashrc"

# Test aws-glue-libs
RUN source "$HOME/.bash_profile" \
    && cd aws-glue-libs \
    && gluepytest || true