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
  + Doc: https://github.com/trep/opentrep/blob/trunk/README.md
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
[OpenTREP README](https://github.com/trep/opentrep/blob/trunk/README.md)
explains how to install the dependencies (_e.g._, Python, Boost, SOCI)
on various Linux distributions and MacOS. Refer to that documentation
for further details.

## Docker images
The [maintained Docker images for that project](docker/)
come with all the necessary pieces of software. They can either be used
_as is_, or used as inspiration for _ad hoc_ setup on other configurations.

## Native environment (outside of Docker)
Refer to the [OpenTREP README](https://github.com/trep/opentrep/blob/trunk/README.md)
for further details.

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
$ docker build -t opentrep/metatrep:fedora --squash docker/fedora/
$ docker push opentrep/metatrep:fedora
$ docker images | grep "^trep"
REPOSITORY           TAG          IMAGE ID        CREATED             SIZE
opentrep/metatrep    centos       9a33eee22a3d    About an hour ago   2.16GB
```

# Usage

```bash
$ ./workspace/install/opentrep/bin/opentrep-indexer -p workspace/install/opentrep/share/opentrep/data/por/optd_por_public_all.csv -t sqlite
$ ./workspace/install/opentrep/bin/opentrep-searcher -t sqlite -q "nce sfo"
Deployment number 0
Xapian database filepath is: /tmp/opentrep/xapian_traveldb0
SQL database type is: sqlite
SQL database connection string is: /tmp/opentrep/sqlite_travel.db0
Log filename is: opentrep-searcher.log
The type of search is: 0
The spelling error distance is: 3
The travel query string is: nce sfo
2 (geographical) location(s) have been found matching your query (`nce sfo'). 0 words were left unmatched.
 [1]: NCE-A-6299418, 8.16788%, Nice Côte d'Azur International Airport, Nice Cote d'Azur International Airport, LFMN, , FRNCE, , 0, 1970-Jan-01, 2999-Dec-31, , NCE|2990440|Nice|Nice, PAC, FR, , France, 427, France, EUR, NA, Europe, 43.6584, 7.21587, S, AIRP, 93, Provence-Alpes-Côte d'Azur, Provence-Alpes-Cote d'Azur, 06, Alpes-Maritimes, Alpes-Maritimes, 062, 06088, 0, 3, 5, Europe/Paris, 1, 2, 1, 2018-Dec-05, , https://en.wikipedia.org/wiki/Nice_C%C3%B4te_d%27Azur_Airport, nce, nce, 8984.66%, 0, 0; name matrix {ar,مطار نيس الريفيرا الفرنسي,de,Flughafen Nizza,en,Nice Côte d'Azur International Airport,en,Nice Airport,es,Niza Aeropuerto,fa,فرودگاه نیس کوت دازور,fr,Aéroport de Nice Côte d'Azur,ja,コート・ダジュール空港,ko,니스 코트다쥐르 공항,ru,Аэропорт Ницца Лазурный Берег,sv,Nice flygplats,wuu,尼斯蓝色海岸机场}
 [2]: SFO-C-5391959, 32.496%, San Francisco, San Francisco, , , USSFO, , 0, 1970-Jan-01, 2999-Dec-31, , SFO|5391959|San Francisco|San Francisco, CA, US, , United States, 91, California, USD, NA, North America, 37.7749, -122.419, P, PPLA2, CA, California, California, 075, City and County of San Francisco, City and County of San Francisco, Z, , 864816, 16, 28, America/Los_Angeles, -8, -7, -8, 2019-Sep-05, SFO, https://en.wikipedia.org/wiki/San_Francisco, sfo, sfo, 35745.6%, 0, 0; name matrix {,San Fransisco,,Yerba Buena,,New Albion,abbr,SF,af,San Francisco,ar,سان فرانسيسكو,arz,سان فرانسيسكو,ast,San Francisco,az,San-Fransisko,be,Сан-Францыска,bg,Сан Франциско,bn,সান ফ্রান্সিস্কো,bo,སན་ཧྥུ་རན་སིས་ཁོ,bpy,সান ফ্রান্সিসকো কাউন্টি,bs,San Francisco,ca,San Francisco,ce,Сан-Франциско,ckb,سان فرانسیسکۆ,cs,San Francisco,cv,Сан-Франциско,cy,San Francisco,da,San Francisco,de,San Francisco,el,Σαν Φρανσίσκο,en,San Francisco,en,Frisco,eo,San-Francisko,eo,Sanfrancisko,eo,San Francisco,es,San Francisco,et,San Francisco,eu,San Frantzisko,ext,San Franciscu,fa,سان فرانسیسکو,fi,San Francisco,fr,San Francisco,fy,San Fransisko,ga,San Francisco,gan,舊金山,gd,San Francisco,gl,San Francisco,hak,Khiu-kîm-sân,haw,Kapalakiko,hbs,San Francisco,he,סן פרנסיסקו,hi,सैन फ्रांसिस्को,hu,San Francisco,hy,Սան Ֆրանցիսկո,id,San Francisco,is,San Francisco,it,San Francisco,ja,サンフランシスコ,ka,სან-ფრანცისკო,kk,Сан Франсиско,ko,샌프란시스코,ko,샌프란,krc,Сан-Франциско,la,Franciscopolis,lt,San Fransiskas,lv,Sanfrancisko,mhr,Сан-Франциско,mk,Сан Франциско,ml,സാൻ ഫ്രാൻസിസ്കോ,mn,Сан-Франциско,mr,सॅन फ्रान्सिस्को,my,ဆန်ဖရန်စစ္စကိုမြို့,nds,San Francisco,new,स्यान फ्रान्सिस्को,nl,San Francisco,nn,San Francisco,no,San Francisco,os,Сан-Франциско,pa,ਸੈਨ ਫਰਾਂਸਿਸਕੋ,pl,San Francisco,pnb,سان فرانسسکو,post,94102,post,94103,post,94104,post,94105,post,94107,post,94108,post,94109,post,94110,post,94111,post,94112,post,94114,post,94115,post,94116,post,94117,post,94118,post,94119,post,94120,post,94121,post,94122,post,94123,post,94124,post,94125,post,94126,post,94127,post,94129,post,94130,post,94131,post,94132,post,94133,post,94134,post,94137,post,94139,post,94140,post,94141,post,94142,post,94143,post,94144,post,94145,post,94146,post,94147,post,94151,post,94159,post,94160,post,94161,post,94163,post,94164,post,94172,post,94177,post,94188,pt,São Francisco,ro,San Francisco,ro,Сан Франциско,ru,Сан-Франциско,sah,Сан Франсиско,scn,San Franciscu,si,සැන් ෆ්‍රැන්සිස්කෝ,sk,San Francisco,sl,San Francisco,sq,San Francisco,sr,Сан Франциско,sv,San Francisco,ta,சான் பிரான்சிஸ்கோ,te,శాన్ ఫ్రాన్సిస్కో,th,ซานฟรานซิสโก,tl,San Francisco,tl,Lungsod ng San Francisco,tr,San Francisco,tt,Сан-Франциско,ug,San Fransisko,uk,Сан-Франціско,uk,Сан-Франциско,ur,سان فرانسسکو,uz,San Fransisko,vep,San Francisko,vi,San Francisco,wuu,旧金山,xmf,სან-ფრანცისკო,yi,סאן פראנציסקא,yue,三藩市,zh,旧金山,zh-CN,旧金山}
```


