# CNTK Dockerfile
#   CPU only
#   No 1-bit SGD

FROM imriss/archlinux

RUN echo 'CNTK CPUOnly on Arch Linux'

MAINTAINER Reza Farrahi <imriss@ieee.org>
LABEL description="CNTK CPUOnly / Arch Linux"

RUN  pacman -Syyu --noconfirm && \
  pacman -S findutils nano vi --needed --noconfirm && \
  pacman-db-upgrade && \
  export editor=nano && \
  pacman -S --needed --noconfirm systemd python python-yaml wget python-pip openssh git curl jq && \
  pip install --upgrade pip && \
  pip install simplejson

# DDADD https://raw.githubusercontent.com/imriss/scylla/master/aur.sh /usr/sbin/aur.sh
# DDADD https://raw.githubusercontent.com/imriss/scylla/master/add-aur.sh /usr/sbin/add-aur
ADD ./aur.sh /usr/sbin/aur.sh
ADD ./add-aur.sh /usr/sbin/add-aur
RUN chmod u+x /usr/sbin/aur.sh
RUN chmod u+x /usr/sbin/add-aur
RUN add-aur docker

RUN su docker -c 'pacaur -S --needed --noprogressbar --noedit --noconfirm base-devel gcc-fortran expect cmake \
    ffmpeg jasper libjpeg-turbo libpng lapacke libtiff protobuf python-protobuf unzip automake libtool autoconf \
    subversion apr apr-util serf m4 jdk7-openjdk openmpi libzip opencv blas boost boost-libs swig '

# ENV PATH /usr/local/mpi/bin:$PATH
ENV LD_LIBRARY_PATH /usr/lib/openmpi/:$LD_LIBRARY_PATH
ENV LD_LIBRARY_PATH /usr/local/lib:$LD_LIBRARY_PATH
ENV LD_LIBRARY_PATH /usr/lib:$LD_LIBRARY_PATH

# Install CNTK custom MKL
RUN CNTK_CUSTOM_MKL_VERSION=3 && \
    mkdir /usr/local/CNTKCustomMKL && \
    wget --no-verbose -O - https://www.cntk.ai/mkl/CNTKCustomMKL-Linux-$CNTK_CUSTOM_MKL_VERSION.tgz | \
    tar -xzf - -C /usr/local/CNTKCustomMKL

WORKDIR /cntk

# Build CNTK
RUN git clone --depth=1 -b master https://github.com/Microsoft/CNTK.git . && \
    CONFIGURE_OPTS="" && \
    git submodule update --init Source/Multiverso && \
    sed -i "s,libprotobuf.a,libprotobuf.so,g" Makefile && \ 
    sed -i "s,libprotobuf.a,libprotobuf.so,g" configure && \
    sed -i "s,    template CNTK_API const float\* NDArrayView::DataBuffer,    "\
"template CNTK_API const TensorView<float>* NDArrayView::GetTensorView<float>() "\
"const;"$"\n    template CNTK_API const TensorView<double>* NDArrayView::GetTensorView<double>() "\
"const;"$"\n\n    template CNTK_API const float\* NDArrayView::DataBuffer,g" Source/CNTKv2LibraryDll/NDArrayView.cpp && \
    mkdir -p build-mkl/cpu/release && \
    cd build-mkl/cpu/release && \
    ../../../configure $CONFIGURE_OPTS --with-mkl=/usr/local/CNTKCustomMKL && \
    make -j"$(nproc)" all && \
    cd ../../.. && \    
    mkdir -p build/cpu/release && \
    cd build/cpu/release && \
    ../../../configure $CONFIGURE_OPTS --with-openblas=/usr/local/openblas && \
    make -j"$(nproc)" all 



RUN cd Examples/Image/DataSets/CIFAR-10 && \
    python install_cifar10.py && \
    cd ../../../..

RUN cd Examples/Image/DataSets/MNIST && \
    python install_mnist.py && \
    cd ../../../..

RUN pacman -Scc --noconfirm && \
  rm /var/cache/pacman/pkg/* && \
  paccache -rf && \
  rm -rd /home/docker/sandground && \
  echo `du /usr/lib -hd 1`

ENV PATH=/cntk/build/cpu/release/bin:$PATH PYTHONPATH=/cntk/bindings/python LD_LIBRARY_PATH=/cntk/bindings/python/cntk/libs:$LD_LIBRARY_PATH
