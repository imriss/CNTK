# CNTK Dockerfile
#   CPU only
#   No 1-bit SGD

FROM imriss/archlinux

RUN echo 'CNTK CPUOnly on Arch Linux'

MAINTAINER Reza Farrahi <imriss@ieee.org>
LABEL description="CNTK CPUOnly / Arch Linux"

RUN  pacman -Syyu --noconfirm \
  && pacman -S findutils nano vi --needed --noconfirm \
  && pacman-db-upgrade \
  && export editor=nano \
  && pacman -S --needed --noconfirm systemd python python-yaml wget python-pip openssh git curl jq \
  && pip install --upgrade pip \
  && pip install simplejson

# DDADD https://raw.githubusercontent.com/imriss/scylla/master/aur.sh /usr/sbin/aur.sh
# DDADD https://raw.githubusercontent.com/imriss/scylla/master/add-aur.sh /usr/sbin/add-aur
ADD ./aur.sh /usr/sbin/aur.sh
ADD ./add-aur.sh /usr/sbin/add-aur
RUN chmod u+x /usr/sbin/aur.sh
RUN chmod u+x /usr/sbin/add-aur
RUN add-aur docker

RUN su docker -c 'pacaur -S --needed --noprogressbar --noedit --noconfirm gcc-fortran expect'


RUN pacman -Scc --noconfirm \
  && rm /var/cache/pacman/pkg/* \
  && paccache -rf \
  && rm -rd /home/docker/sandground \
  && echo `du /usr/lib -hd 1`

ENV PATH=/cntk/build/cpu/release/bin:$PATH PYTHONPATH=/cntk/bindings/python LD_LIBRARY_PATH=/cntk/bindings/python/cntk/libs:$LD_LIBRARY_PATH
