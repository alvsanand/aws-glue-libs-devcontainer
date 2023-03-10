FROM mcr.microsoft.com/vscode/devcontainers/base:ubuntu-22.04

SHELL ["/bin/bash", "-c"]

USER root

ARG USERNAME=vscode

RUN apt-get update \
    && apt-get install -y curl \
                          git \
                          gnupg2 \
                          jq \
                          software-properties-common \
                          telnet \
                          unzip \
                          vim \
                          wget \
                          zip

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
RUN curl -s "https://get.sdkman.io?rcupdate=false" | bash

ARG JDK_VERSION="11"
ARG SCALA_VERSION="2.12.17"
ENV SCALA_BINARY_VERSION="2.12"
ARG MAVEN_VERSION="3.8.7"
ARG SBT_VERSION="1.8.0"

RUN source "${HOME}/.sdkman/bin/sdkman-init.sh" \
    && FINAL_JDK_VERSION=$(sdk list java | sed -En 's/.+ ('${JDK_VERSION}'(.+)-amzn).*/\1/p') \
    && sdk install java "$FINAL_JDK_VERSION" \
    && sdk install scala "$SCALA_VERSION" \
    && sdk install sbt "$SBT_VERSION" \
    && sdk install maven "$MAVEN_VERSION"

# Install aws-glue-libs
ARG AWS_GLUE_SPARK_URL="https://aws-glue-etl-artifacts.s3.amazonaws.com/glue-4.0/spark-3.3.0-amzn-1-bin-3.3.3-amzn-0.tgz"
ARG AWS_GLUE_VERSION="4.0"

RUN wget -qO- "$AWS_GLUE_SPARK_URL" | tar zxf - 2>/dev/null \
    && mv build/spark spark && rm -Rf build

RUN wget -qO- "https://github.com/awslabs/aws-glue-libs/archive/refs/tags/v$AWS_GLUE_VERSION.tar.gz" | tar zxf - 2>/dev/null \
    && mv aws-glue-libs-* aws-glue-libs \
    && python3 -m pip install boto3 pytest

# Configure profile
RUN echo 'alias python=python3' >> "${HOME}/.bashrc"
RUN echo 'export PATH="${HOME}/aws-glue-libs/bin:${HOME}/spark/bin:${PATH}":${HOME}/.local/bin' >> "${HOME}/.bashrc"
RUN echo 'export SPARK_HOME="${HOME}/spark"' >> "${HOME}/.bashrc"
RUN echo '[[ -s "$HOME/.sdkman/bin/sdkman-init.sh" ]] && source "$HOME/.sdkman/bin/sdkman-init.sh"' >> "${HOME}/.bashrc"

# Init aws-glue-libs
ADD .devcontainer/pom.xml "${HOME}/aws-glue-libs/pom.xml"
ADD .devcontainer/glue-setup.sh "${HOME}/aws-glue-libs/bin/glue-setup.sh"

RUN bash -l -i -c 'source "$HOME/.bashrc" && bash "$HOME/aws-glue-libs/bin/glue-setup.sh"'