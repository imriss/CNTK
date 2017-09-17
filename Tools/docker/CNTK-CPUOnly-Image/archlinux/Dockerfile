# CNTK Dockerfile
#   CPU only
#   No 1-bit SGD
FROM imriss/archlinux
RUN echo 'CNTK CPUOnly on Arch Linux'
MAINTAINER Reza Farrahi <imriss@ieee.org>
LABEL description="CNTK CPUOnly / Arch Linux"

RUN  pacman -Syyu --noconfirm && \
  pacman -S findutils nano vi --noconfirm --needed && \
  pacman-db-upgrade && \
  export editor=nano && \
  pacman -S --noconfirm --needed systemd python python-yaml wget python-pip openssh git curl jq yajl perl expac \
  gcc gcc-libs clang && \
  pip install --upgrade pip && \
  pip install simplejson

# DDADD https://raw.githubusercontent.com/imriss/scylla/master/aur.sh /usr/sbin/aur.sh
# DDADD https://raw.githubusercontent.com/imriss/scylla/master/add-aur.sh /usr/sbin/add-aur
ADD ./aur.sh /usr/sbin/aur.sh
ADD ./add-aur.sh /usr/sbin/add-aur
RUN chmod u+x /usr/sbin/aur.sh
RUN chmod u+x /usr/sbin/add-aur
RUN export "PATH=/usr/bin/core_perl:$PATH" && \
    add-aur docker



RUN su docker -c 'pacaur -S --noprogressbar --noedit --noconfirm --needed base-devel expect cmake gcc-fortran \
    ffmpeg jasper libjpeg-turbo libpng lapacke libtiff protobuf python-protobuf unzip automake libtool autoconf \
    subversion apr apr-util serf m4 jdk7-openjdk openmpi python2-mpi4py libzip opencv openblas-lapack boost boost-libs swig \
    anaconda swig gcc5 '
    
# RUN su docker -c 'pacaur -S --noprogressbar --noedit --noconfirm kaldi '

# RUN export "PATH=/usr/bin/core_perl:$PATH" && \
#     cd /home/docker/sandground && \
#     su docker -c 'mkdir kaldi' && cd kaldi && echo `ls -la` && \
#     wget https://aur.archlinux.org/cgit/aur.git/snapshot/kaldi.tar.gz && su docker -c 'tar -xvf kaldi.tar.gz' && cd kaldi && \
#     sed -i "s,cd \$srcdir\/\$pkgname\/tools,export CXXFLAGS=\"\$(printf '%s' \"\$CXXFLAGS\" | sed 's/-fno-plt//')\"\
# "$"\n        cd \$srcdir\/\$pkgname\/tools,g" PKGBUILD && \
#     sed -i "s,cd \$srcdir\/\$pkgname,cd \$srcdir\/\$pkgnamefull,g" PKGBUILD && \
#     sed -i "s,cd \"\${pkgname}\",cd \"\${pkgnamefull}\",g" PKGBUILD && \
#     sed -i "s,pkgname='kaldi',pkgname='kaldi'\
#     "$"\npkgnamefull='kaldi-c024e8aa0a727bf76c91a318f76a1f8b0b59249e'\
#     "$"\nKALDI_VERSION='c024e8aa',g" PKGBUILD && \
#     sed -i "s,source=(\"\${pkgname}::git+\${url}\",\
# source=(\"https:\/\/github.com\/kaldi-asr\/kaldi\/archive\/\$KALDI_VERSION.tar.gz\",g" PKGBUILD && \
#   sed -i 's~check_dependencies\.sh~check_dependencies\.sh\n        sed -i "s,rm -rf sctk \\\&\\\& ln -s sctk-2\.4\.9 sctk,rm -rf sctk \\\&\\\& ln -s sctk-2\.4\.9 sctk\\ \
# \\n\\tfind \. -type f | xargs sed -i  \\'\''s\/-fno-plt\/\/\\'\'',g" Makefile~g' PKGBUILD && \
#     su docker -c 'makepkg -si' && \
#     pacman -U --noconfirm kaldi-5.2.r7528.5e91c5d2e-1-x86_64.pkg.tar
    
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
    cp /cntk/Scripts/install/linux/conda-linux-cntk-py36-environment.yml /tmp/ && \
    /opt/anaconda/bin/conda env create -p /opt/anaconda/envs/cntk-py36/ --file /tmp/conda-linux-cntk-py36-environment.yml 
ENV PATH /opt/anaconda/envs/cntk-py36/bin:$PATH
ADD activate-cntk ./activate-cntk
RUN CONFIGURE_OPTS="--with-py36-path=/opt/anaconda/envs/cntk-py36" && \
    git submodule update --init Source/Multiverso && \
    git submodule sync --recursive && \
    git submodule update --init --recursive && \
    git submodule sync --recursive && \
    git submodule update --init --recursive && \
    sed -i "s,libprotobuf.a,libprotobuf.so,g" Makefile && \ 
    sed -i "s,libprotobuf.a,libprotobuf.so,g" configure && \
    sed -i "s,QWordVal qval = valQ.Quantize,QWordVal qval = valQ.template Quantize,g" Source/Math/ColumnQuantizer.h && \
    sed -i "s,libmpi.so.12,libmpi.so,g" bindings/python/cntk/train/distributed.py && \
    mkdir -p build-mkl/cpu/release && \
    cd build-mkl/cpu/release && \
    ../../../configure $CONFIGURE_OPTS --with-mkl=/usr/local/CNTKCustomMKL && \
    make -j"$(nproc)" all
    
    # cd ../../.. && \    
    # mkdir -p build/cpu/release && \
    # cd build/cpu/release && \
    # ../../../configure $CONFIGURE_OPTS --with-openblas=/usr/pkgs/openblas-0.2.19-2 && \
    # make -j"$(nproc)" all 

# RUN cd Examples/Image/DataSets/CIFAR-10 && \
#     python install_cifar10.py && \
#     cd ../../../..

# RUN cd Examples/Image/DataSets/MNIST && \
#    python install_mnist.py && \
#    cd ../../../..

# RUN export CNTK_COMPONENT_VERSION=2.0 && \
  # cd /cntk/bindings/python && \
  # python ./setup.py build_ext --inplace --force 

RUN pacman -Scc --noconfirm && \
  rm /var/cache/pacman/pkg/* && \
  paccache -rf && \
  rm -rd /home/docker/sandground && \
  rm -rd /home/docker/.cache && \
  rm -rd /opt/anaconda/lib && \
  rm -rd /opt/anaconda/pkgs && \
  rm -rd /cntk/build-mkl/cpu/release/.build && \
  echo `du /usr/lib -hd 1`

ENV PATH=/cntk/build-mkl/cpu/release/bin:$PATH PYTHONPATH=/cntk/bindings/python LD_LIBRARY_PATH=/cntk/bindings/python/cntk/libs:$LD_LIBRARY_PATH
CMD ["source", "/cntk/activate-cntk"]
