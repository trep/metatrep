MetaTREP - Umbrella project for OpenTREP
========================================

[![Docker Repository on Quay](https://quay.io/repository/trep/metatrep/status "Docker Repository on Quay")](https://quay.io/repository/trep/metatrep)

Development helpers for the OpenTREP components.

The `metatrep` project itself is an umbrella, allowing to drive all
the other components from a central local directory, namely `workspace/`.
One can then interact with any specific component directly by jumping
(`cd`-ing) into the corresponding directory. Software code can be edited
and committed directly from that component sub-directory.

[Docker images, hosted on Docker Cloud](https://cloud.docker.com/u/opentrep/repository/docker/opentrep/metatrep),
are provided for convenience reason, avoiding the need to set up
a proper development environment: they provide a ready-to-use,
ready-to-develop, ready-to-contribute environment. Enjoy!

# References
* Open Travel REquest Parser (OpenTREP):
  + Source code on GitHub: https://github.com/trep
  + Docker Cloud repository: https://cloud.docker.com/u/opentrep/repository/docker/opentrep/metatrep
  + Web applications:
    - https://transport-search.org
    - https://www2.transport-search.org

# Run the Docker image
* As a quick starter, some test cases can be launched from the
  [one of Docker images (_eg_, CentOS, Ubuntu or Debian)](docker/):
```bash
$ docker run --rm -it opentrep/metatrep:centos bash
[build@c..5 metatrep]$ cd workspace/build/trep
[build@c..5 trep (master)]$ make check
[build@c..5 trep (master)]$ exit
```

# Meta-build
The [Docker images](https://cloud.docker.com/u/opentrep/repository/docker/opentrep/metatrep)
come with all the dependencies already installed. If there is a need,
however, for some more customization (for instance, install some
other software products such as [Kafka](https://kafka.apache.org) or
[ElasticSearch](http://elasticsearch.com)), this section describes
how to get the end-to-end travel market simulator up
and running on a native environment (as opposed to within
a Docker container).

An alternative is to develop your own Docker image from the
[ones provided by that project](https://cloud.docker.com/u/opentrep/repository/docker/opentrep/metatrep).
You would typically start the `Dockerfile` with
`FROM opentrep/metatrep:<linux-distribution>`.

## Installation of dependencies (if not using the Docker image)
C++, Python and Ruby are needed in order to build
and run the various components of that project.

## Docker images
The [maintained Docker images for that project](docker/)
come with all the necessary pieces of software. They can either be used
_as is_, or used as inspiration for _ad hoc_ setup on other configurations.

## Native environment (outside of Docker)

### CentOS/RedHat
* Install [EPEL for CentOS/RedHat](https://fedoraproject.org/wiki/EPEL):
```bash
$ sudo rpm --import http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-7
$ sudo yum -y install epel-release
```

* Install a few useful packages:
```bash
$ sudo yum -y install less htop net-tools which sudo man vim \
        git-all wget curl file bash-completion keyutils Lmod \
        zlib-devel bzip2-devel gzip tar rpmconf yum-utils \
        gcc gcc-c++ cmake cmake3 m4 \
		lcov cppunit-devel \
        zeromq-devel czmq-devel cppzmq-devel \
        boost-devel xapian-core-devel openssl-devel libffi-devel \
        mpich-devel openmpi-devel \
        readline-devel sqlite-devel mariadb-devel \
        soci-mysql-devel soci-sqlite3-devel \
        libicu-devel protobuf-devel protobuf-compiler \
        python-devel \
        python34 python34-pip python34-devel \
        python2-django mod_wsgi \
        geos-devel geos-python \
        doxygen ghostscript "tex(latex)" texlive-epstopdf-bin \
		rake rubygem-rake ruby-libs
```

### MacOS
* Install [Homebrew](https://brew.sh), if not already done:
```bash
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

* SOCI. As of mid 2019, SOCI 4.0 has still not been released,
  and `soci-mysql` is no longer available. Hence, SOCI must be built
  from the sources. The following shows how to do that:
```bash
$ mkdir -p ~/dev/infra/soci && cd ~/dev/infra/soci
$ git clone https://github.com/SOCI/soci.git
$ cd soci
$ mkdir build && cd build
$ cmake -DSOCI_TESTS=OFF ..
$ make
$ sudo make install
```

* C++, Python and Ruby:
```bash
$ brew install boost boost-python boost-python3 cmake libedit \
  sqlite mysql icu4c protobuf protobuf-c zeromq doxygen
$ wget https://raw.githubusercontent.com/zeromq/cppzmq/master/zmq.hpp -O /usr/local/include/zmq.hpp
$ brew install readline homebrew/portable-ruby/portable-readline
$ brew install pyenv
$ brew install rbenv ruby-build
```

### All
* Install Python [Pyenv] and [`pipenv`](https://pipenv.readthedocs.io):
```bash
$ git clone https://github.com/pyenv/pyenv.git ${HOME}/.pyenv
$ cat >> ~/.bashrc << _EOF

# Python pyenv
export PYENV_ROOT="\${HOME}/.pyenv"
export PATH="\${PYENV_ROOT}/bin:\${PATH}"

if command -v pyenv 1>/dev/null 2>&1; then
    eval "\$(pyenv init -)"
fi
_EOF
$ source ${HOME}/.bashrc && pyenv install 3.7.2
$ pip3 install --user -U pipenv
```

## Clone the Git repository
The following operation needs to be done only on a native environment (as
opposed to within a Docker container).
The Docker image indeed comes with that Git repository already cloned and built.
In the following, `<linux-distrib>` may be one of `centos`, `ubuntu` or `debian`.
```bash
$ mkdir -p ~/dev/geo && cd ~/dev/geo
$ git clone https://github.com/trep/metatrep.git
$ cd metatrep
$ cp docker/<linux-distrib>/resources/metatrep.yaml.sample metatrep.yaml
$ rake clone
$ rake checkout
$ rake offline=true info
```

## Interactive build with `rake`
That operation may be done either from within the Docker container,
or in a native environment (on which the dependencies have been installed).

As a reminder, to enter into the container, just type
`docker run --rm -it tvlsim/metasim:<linux-distrib> bash`, and `exit`
to leave it (`<linux-distrib>` may be one of `centos`, `ubuntu` or `debian`).

The following sequence of commands describes how to build, test and deliver
the artefacts of all the components, so that a full simulation may be performed:
```bash
$ cd ~/dev/sim/metasim
$ rm -rf workspace/build workspace/install
$ rake offline=true configure
$ rake offline=true install
$ rake offline=true test
$ rake offline=true dist
```

## Interacting with a specific project
Those operations may be done either from within the Docker container,
or in a native environment (on which the dependencies have been installed).

As a reminder, to enter into the container, just type
`docker run --rm -it opentrep/metatrep:<linux-distrib> bash`, and `exit`
to leave it (`<linux-distrib>` may be one of `centos`, `ubuntu` or `debian`).

```bash
$ cd ~/dev/geo/metatrep
$ cd workspace/src/opentrep
$ vi opentrep/bom/Place.cpp
$ git add opentrep/bom/Place.cpp
$ cd ../../build/opentrep
$ make check && make install
$ # If all goes well at the component level, re-build the full simulator
$ cd ~/dev/geo/metatrep
$ rake offline=true test
$ # If all goes well at the integration level
$ cd workspace/src/opentrep
$ git commit -m "[Dev] Fixed issue #76: C++-20 compatibility"
$ cd -
```

## Batched build and Docker image generation
If the Docker images need to be re-built, the following commands explain
how to do it:
```bash
$ mkdir -p ~/dev/geo && cd ~/dev/geo
$ git clone https://github.com/trep/metatrep.git
$ cd metatrep
$ docker build -t opentrep/metatrep:centos --squash docker/centos/
$ docker push opentrep/metatrep:centos
$ docker build -t opentrep/metatrep:debian --squash docker/debian/
$ docker push opentrep/metatrep:debian
$ docker build -t opentrep/metatrep:ubuntu --squash docker/ubuntu/
$ docker push opentrep/metatrep:ubuntu
$ docker images | grep "^trep"
REPOSITORY           TAG          IMAGE ID        CREATED             SIZE
opentrep/metatrep    centos       9a33eee22a3d    About an hour ago   2.16GB
```


