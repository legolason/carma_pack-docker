FROM ubuntu

ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update && apt-get install -y --no-install-recommends apt-utils

#Runit
RUN apt-get install -y runit 
CMD /usr/sbin/runsvdir-start

#SSHD
RUN apt-get install -y openssh-server && \
    mkdir -p /var/run/sshd && \
    echo 'root:root' |chpasswd
RUN sed -i "s/session.*required.*pam_loginuid.so/#session    required     pam_loginuid.so/" /etc/pam.d/sshd
RUN sed -i "s/PermitRootLogin without-password/#PermitRootLogin without-password/" /etc/ssh/sshd_config

#Utilities
RUN apt-get install -y vim less net-tools inetutils-ping curl git telnet nmap socat dnsutils netcat tree htop unzip sudo software-properties-common

#Required by Python packages
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y build-essential python-dev python-pip liblapack-dev libatlas-dev gfortran libfreetype6 libfreetype6-dev libpng12-dev python-lxml libyaml-dev g++ libffi-dev

#0MQ
RUN cd /tmp && \
    wget http://download.zeromq.org/zeromq-4.0.3.tar.gz && \
    tar xvfz zeromq-4.0.3.tar.gz && \
    cd zeromq-4.0.3 && \
    ./configure && \
    make install && \
    ldconfig

#Upgrade pip
RUN pip install -U setuptools
RUN pip install -U pip

#matplotlib needs latest distribute
RUN pip install -U distribute

#IPython
RUN pip install ipython
ENV IPYTHONDIR /ipython
RUN mkdir /ipython && \
    ipython profile create nbserver

#NumPy
RUN pip install numpy

#Pandas
RUN pip install pandas

#Optional
RUN pip install scipy
RUN apt-get install pkg-config
RUN pip install matplotlib

#Add runit services
#ADD sv /etc/services

#Need to install acor for carma_pack
RUN pip install acor


RUN apt-get update \
 && DEBIAN_FRONTEND=noninteractive apt-get install -y build-essential wget libbz2-dev \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

ARG boost_version=1.64.0
ARG boost_dir=boost_1_64_0
ENV boost_version ${boost_version}

RUN wget https://dl.bintray.com/boostorg/release/${boost_version}/source/${boost_dir}.tar.gz \
    && tar xfz ${boost_dir}.tar.gz \
    && rm ${boost_dir}.tar.gz \
    && cd ${boost_dir} \
    && ./bootstrap.sh \
    && ./b2 --prefix=/usr -j 4 link=shared runtime-link=shared install \
    && cd .. && rm -rf ${boost_dir} && ldconfig

RUN apt-get update

CMD bash

#install armadillo libraries for carma_pack
RUN apt-get install -y libarmadillo-dev

RUN apt-get install -y libboost-python-dev

#set environment variales for carma_pack install
ENV BOOST_DIR=/usr/include/boost
ENV ARMADILLO_DIR=/usr/lib
ENV NUMPY_DIR=/usr/local/lib/python2.7/dist-packages

RUN apt-get install -y git
RUN apt-get install -y python-tk

ENV CARMA_DIR=/carma/carma_pack

RUN mkdir /carma && git clone https://github.com/brandonckelly/carma_pack.git $CARMA_DIR && cd ${CARMA_DIR}/src && ls && echo 'next ls' && ls .. && python setup.py install && ldconfig

RUN pip install astroML
RUN pip install astroML_addons
RUN pip install pyfits

ENV LANG=C.UTF-8 LC_ALL=C.UTF-8

RUN apt-get update --fix-missing && apt-get install -y wget bzip2 ca-certificates \
    libglib2.0-0 libxext6 libsm6 libxrender1 \
    git mercurial subversion

RUN echo 'export PATH=/opt/conda/bin:$PATH' > /etc/profile.d/conda.sh && \
    wget --quiet https://repo.continuum.io/archive/Anaconda3-4.4.0-Linux-x86_64.sh -O ~/anaconda.sh && \
    /bin/bash ~/anaconda.sh -b -p /opt/conda && \
    rm ~/anaconda.sh

RUN apt-get install -y curl grep sed dpkg && \
    TINI_VERSION=`curl https://github.com/krallin/tini/releases/latest | grep -o "/v.*\"" | sed 's:^..\(.*\).$:\1:'` && \
    curl -L "https://github.com/krallin/tini/releases/download/v${TINI_VERSION}/tini_${TINI_VERSION}.deb" > tini.deb && \
    dpkg -i tini.deb && \
    rm tini.deb && \
    apt-get clean

ENV PATH /opt/conda/bin:$PATH
ENV MKL_PATH=/opt/intel

RUN apt-get install -y \
    cmake \
    cpio \
    man 

# Install MKL
RUN cd /tmp && \
  wget http://registrationcenter-download.intel.com/akdlm/irc_nas/tec/11306/l_mkl_2017.2.174.tgz && \
  tar -xzf l_mkl_2017.2.174.tgz && \
  cd l_mkl_2017.2.174 && \
  sed -i 's/ACCEPT_EULA=decline/ACCEPT_EULA=accept/g' silent.cfg && \
  ./install.sh -s silent.cfg && \
  cd .. && \
  rm -rf *
RUN echo "/opt/intel/mkl/lib/intel64" >> /etc/ld.so.conf.d/intel.conf && \
  ldconfig && \
  echo ". /opt/intel/bin/compilervars.sh intel64" >> /etc/bash.bashrc

## The nloptr package is needed by lme4, and it itself needs to download the
## NLopt code from http://ab-initio.mit.edu/wiki/index.php/NLopt, which is
## unstable. Hence we put this upfront, so that we fail fast on this step,
## which makes it easier to redo.

RUN cd /tmp && \
	wget http://ab-initio.mit.edu/nlopt/nlopt-2.4.1.tar.gz && \
	tar zxf nlopt-2.4.1.tar.gz && \
	cd nlopt-2.4.1 && \
	./configure --with-pic && \
	make -j$(nproc) && \
	make install && \
	rm -rf /tmp/nlopt-*

RUN conda install pytest && conda install python=2.7 && conda install libgcc
RUN pip install Cython
RUN pip install future
RUN pip install fitsio
RUN pip install pytest
RUN pip install gatspy
RUN pip install multi_key_dict
RUN apt-get update
RUN apt-get install -y libgl1-mesa-glx libgl1-mesa-dev
ENV KALI_DIR=/carma/kali

##RUN git clone https://github.com/AstroVPK/kali.git $KALI_DIR && cd ${KALI_DIR} && python setup.py install && ldconfig

RUN apt-get install -qqy x11-apps
ENV DISPLAY :0
RUN conda update libgcc
RUN dbus-uuidgen >/etc/machine-id
RUN apt-get install -y dvipng texlive-latex-extra
ENV LC_ALL=C