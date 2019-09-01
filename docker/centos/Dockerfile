#
# https://github.com/trep/metatrep/blob/master/docker/centos/Dockerfile
# Umbrella project for Open Travel Request Parser (OpenTREP)
#
FROM cpppythondevelopment/base:centos7
MAINTAINER Denis Arnaud <denis.arnaud_github at m4x dot org>

# Docker build time environment variables
ENV container docker
ENV HOME /home/build
ENV LANGUAGE en_US:en
ENV LANG en_US.UTF-8
ENV LC_ALL $LANG
ENV TREP_DIR $HOME/dev/geo/opentrep-meta

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

