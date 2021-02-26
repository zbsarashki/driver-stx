FROM centos/tools

# Add StarlingX Kernel-rt repo
RUN echo -e $'\
[stx-kernel-rt]\n\
name=StarlingX\n\
baseurl=http://mirror.starlingx.cengn.ca/mirror/starlingx/release/4.0.1/centos/distro/outputs/RPMS/rt/\n\
enabled=1\n\
gpgcheck=0' \
> /etc/yum.repos.d/STX-x86_64-kernel-rt.repo

#RUN yum install -y xmlto asciidoc openssl hmaccalc python-devel newt-devel pesign \
#	elfutils-devel binutils-devel audit-libs-devel java-devel \
#	pciutils-devel ncurses-devel python-docutils perl-ExtUtils-Embed wget \
#	which net-tools bc numactl-devel ca-certificates curl glibc.i686 \
#        kmod gcc ca-certificates make && \
#	rm -rf /var/cache/yum/*

RUN yum install -y \
	ca-certificates \
	curl \
	gcc \
	glibc.i686 \
	make \
	elfutils-devel \
	kmod && \
	rm -rf /var/cache/yum/*

RUN yum install -y kernel-rt-devel kernel-rt-headers kernel-rt-core

RUN curl -fsSL -o /usr/local/bin/donkey https://github.com/3XX0/donkey/releases/download/v1.1.0/donkey && \
    curl -fsSL -o /usr/local/bin/extract-vmlinux https://raw.githubusercontent.com/torvalds/linux/master/scripts/extract-vmlinux && \
    chmod +x /usr/local/bin/donkey /usr/local/bin/extract-vmlinux
#
##ARG BASE_URL=http://us.download.nvidia.com/XFree86/Linux-x86_64
ARG BASE_URL=https://us.download.nvidia.com/tesla
ARG DRIVER_VERSION=450.80.02
ENV DRIVER_VERSION=$DRIVER_VERSION
#
## Install the userspace components and copy the kernel module sources.
RUN cd /tmp && \
    curl -fSsl -O $BASE_URL/$DRIVER_VERSION/NVIDIA-Linux-x86_64-$DRIVER_VERSION.run && \
    sh NVIDIA-Linux-x86_64-$DRIVER_VERSION.run -x && \
    cd NVIDIA-Linux-x86_64-$DRIVER_VERSION* && \
    ./nvidia-installer --silent \
                       --no-kernel-module \
                       --install-compat32-libs \
                       --no-nouveau-check \
                       --no-nvidia-modprobe \
                       --no-rpms \
                       --no-backup \
                       --no-check-for-alternate-installs \
                       --no-libglx-indirect \
                       --no-install-libglvnd \
                       --x-prefix=/tmp/null \
                       --x-module-path=/tmp/null \
                       --x-library-path=/tmp/null \
                       --x-sysconfig-path=/tmp/null && \
    mkdir -p /usr/src/nvidia-$DRIVER_VERSION && \
    mv LICENSE mkprecompiled kernel /usr/src/nvidia-$DRIVER_VERSION && \
    sed '9,${/^\(kernel\|LICENSE\)/!d}' .manifest > /usr/src/nvidia-$DRIVER_VERSION/.manifest \ 
    && rm -rf /tmp/*

COPY nvidia-driver /usr/local/bin

WORKDIR /usr/src/nvidia-$DRIVER_VERSION

ARG PUBLIC_KEY=empty
COPY ${PUBLIC_KEY} kernel/pubkey.x509

ARG PRIVATE_KEY
ARG KERNEL_VERSION=latest
ENV IGNORE_PREEMPT_RT_PRESENCE=1 

RUN mkdir -p /run/nvidia

# Compile the kernel modules and generate precompiled packages for use by the nvidia-installer.
#RUN yum makecache -y && \
#       IGNORE_PREEMPT_RT_PRESENCE=1 nvidia-driver update -k $(uname -r) -t builtin ${PRIVATE_KEY:+"-s ${PRIVATE_KEY}"} && \
#    rm -rf /var/cache/yum/*
#ENTRYPOINT ["nvidia-driver", "init"]
