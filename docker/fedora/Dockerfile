#
# https://github.com/trep/metatrep/blob/master/docker/fedora/Dockerfile
# Umbrella project for Open Travel Request Parser (OpenTREP)
#
FROM fedora:rawhide
MAINTAINER Denis Arnaud <denis.arnaud_github at m4x dot org>

# Docker build time environment variables
ENV container docker
ENV HOME /home/build
ENV LANGUAGE en_US:en
ENV LANG en_US.UTF-8
ENV LC_ALL $LANG
ENV TREP_DIR $HOME/dev/geo/opentrep-meta

# Upgrade the OS
RUN dnf -y update && \
  dnf -y install rpmconf dnf-utils htop wget curl less net-tools \
    whois screen bzip2 dos2unix man man-pages keychain \
    sudo ftp rsync vim git-all patch mutt links lynx \
    fedpkg \
    gcc-c++ cmake make zlib-devel bzip2-devel libffi-devel \
    doxygen \
    readline-devel openssl-devel sqlite-devel mariadb-devel \
    python3 python3-devel python3-pip \
    boost-python3-devel xapian-core-devel \
    soci-mysql-devel soci-sqlite3-devel readline-devel \
    sqlite-devel mariadb-devel libicu-devel \
    protobuf-devel protobuf-compiler

# Create the `build` user (for the development activities)
RUN adduser build
RUN echo "build ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/build && \
    chmod 0440 /etc/sudoers.d/build

# Configure SSH
RUN mkdir -p $HOME/.ssh && chmod 700 $HOME/.ssh

# Set up the packaging environment for the `build` user
ADD resources/bashrc $HOME/.bashrc
ADD resources/gitconfig $HOME/.gitconfig
ADD resources/vimrc $HOME/.vimrc
RUN chmod 640 $HOME/.bashrc $HOME/.gitconfig $HOME/.vimrc
RUN chown -R build.build $HOME

# Switch to the `build` user
WORKDIR $HOME
USER build

# Git prompt
RUN git clone https://github.com/magicmonty/bash-git-prompt.git $HOME/.bash-git-prompt --depth=1

# Python Pyenv and pipenv
RUN git clone https://github.com/pyenv/pyenv.git $HOME/.pyenv
ENV PATH $HOME/.pyenv/bin:$HOME/.pyenv/shims:$PATH
RUN pyenv install 3.7.2 && \
    pyenv global 3.7.2 && \
    pip install -U pip && \
    pip install -U pipenv && \
    pyenv global system

# Switch to the `root` user for complementary installation and configuration
USER root
ADD resources/ssh-config $HOME/.ssh/config
RUN chown build.build $HOME/.ssh/config

# Switch to the `build` user
USER build

# Configure SSH
RUN ssh-keyscan github.com > $HOME/.ssh/known_hosts
RUN chmod 600 $HOME/.ssh/config $HOME/.ssh/known_hosts

# Prepare the build environment
RUN mkdir -p $TREP_DIR
WORKDIR $TREP_DIR

ADD resources/Rakefile resources/metatrep.yaml.sample $TREP_DIR/
RUN mv $TREP_DIR/metatrep.yaml.sample $TREP_DIR/metatrep.yaml

# Build and deliver all the components
RUN rake checkout && \
	rake offline=true configure && \
	rake offline=true install && \
	rake offline=true dist

# Entry point
CMD ["/bin/bash"]

