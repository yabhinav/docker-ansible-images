# 
# Dockerfile for building Ansible image from source for CentOS 6, with as few additional software as possible.
#
# Version  0.1.0
#

# pull base OS image
FROM fedora:25

MAINTAINER  yabhinav <yabhinav.github.com>

ENV container docker

# Install systemd 
# @see https://hub.docker.com/_/centos/
RUN \
    (cd /lib/systemd/system/sysinit.target.wants/ || exit; for i in *; do [ "$i" = systemd-tmpfiles-setup.service ] || rm -f "$i"; done); \
    rm -f /lib/systemd/system/multi-user.target.wants/*; \
    rm -f /etc/systemd/system/*.wants/*; \
    rm -f /lib/systemd/system/local-fs.target.wants/*; \
    rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
    rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
    rm -f /lib/systemd/system/basic.target.wants/*; \
    rm -f /lib/systemd/system/anaconda.target.wants/*; \
    \
    \
    echo "===> Installing initscripts to emulate normal OS behavior..."  && \
    dnf -y install initscripts iproute ; \
    \
    \
    echo "===> Installing Ansible prerequisites..."           && \
    dnf -y --best --allowerasing install                         \
                git sudo gcc make python2-dnf                    \
                python python-devel python-pip                   \
                python-keyczar libselinux-python                 \
                python-crypto python-paramiko  openssl-devel  && \
    dnf clean all; \
    \
    \
    echo "===> Installing Ansible ..."             && \
    pip install --upgrade pip                      && \
    pip install --upgrade                             \
        pyyaml jinja2  httplib2 ansible==2.2.0.0   && \
    \
    \
    echo "===> Disabling sudo 'requiretty' setting..."               && \
    sed -i -e 's/^\(Defaults\s*requiretty\)/#--- \1/'  /etc/sudoers;    \
    \
    echo "===> Adding hosts for convenience..."   && \
    mkdir -p /etc/ansible                         && \ 
    echo -e '[local]\nlocalhost ansible_connection=local' > /etc/ansible/hosts; \
    echo -e '[defaults]\nretry_files_enabled = False' > /etc/ansible/ansible.cfg


VOLUME ["/sys/fs/cgroup"]
CMD ["/usr/sbin/init"]
