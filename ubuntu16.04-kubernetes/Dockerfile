FROM ubuntu:16.04
ARG SSHPUBKEY

USER root
RUN mkdir -p /etc/ssh /root/.ssh && chmod 0700 /root/.ssh
RUN echo $SSHPUBKEY > /root/.ssh/authorized_keys && \
    chmod 600 /root/.ssh/authorized_keys

RUN apt-get update && \
    apt-get install -y ca-certificates util-linux openssh-server openssh-client \
                       rng-tools dhcpcd5 kmod linux-virtual \
                       apt-transport-https curl gnupg-agent software-properties-common && \
    apt-get clean

# copy the kernel for later use
RUN rm /vmlinuz && cp /boot/vmlinuz-* /vmlinuz

# install docker and kubernetes
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add - && \
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

RUN add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable" && \
    add-apt-repository "deb https://apt.kubernetes.io/ kubernetes-xenial main"

RUN apt-get update && \
    apt-get install -y docker-ce docker-ce-cli containerd.io \
                       kubelet kubeadm kubectl && \
    apt-get clean && \
    apt-mark hold kubelet kubeadm kubectl

COPY files/init /init
COPY files/scripts/kube-init.sh /root/kube-init.sh

# enable docker
RUN /lib/systemd/systemd-sysv-install enable docker

# set RAMDISK on docker
RUN sed -i -E '/\[Service\]/a \
                    Environment="DOCKER_RAMDISK=true"' \
                /lib/systemd/system/docker.service

# enable root autologin on tty1
RUN cp /lib/systemd/system/getty@.service /lib/systemd/system/autologin@.service && \
    sed -i -E 's/ExecStart=.+/ExecStart=-\/sbin\/agetty -a root %I 38400/' \
        /lib/systemd/system/autologin@.service && \
    ln -sf /lib/systemd/system/autologin@.service /etc/systemd/system/getty.target.wants/getty@tty1.service

# sshd requires /var/run/sshd to exist, but for some reason it doesn't get created
# and if we create it in the docker container, it ends up getting overwritten since it's
# a tmpfs.
RUN sed -i -E '/ExecStartPre=.+/i \
                    ExecStartPre=-/bin/mkdir -p \/var\/run\/sshd' \
                /lib/systemd/system/ssh.service

RUN sed -i 's/root:!/root:*/' /etc/shadow

RUN sed -i '/floppy:x:11:root/d' /etc/group
