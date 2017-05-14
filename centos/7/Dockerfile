# 
# Dockerfile for building Ansible image from source for CentOS 7, with as few additional software as possible.
#
# Multiple stable 2.x.0.0 versions are installed over single image
# to execute playbooks on certain version run command :
#
# docker exec --tty "$(cat ${container_id})" bash -c "source $HOME/.bashrc; workon ansible-2.2.0.0 ansible-playbook --version " 
# 
# Author  : Abhinav Y <https://yabhinav.github.io>
# Version : 0.2.0
#

# Pull Base OS image
FROM centos:7

MAINTAINER  yabhinav <yabhinav.github.com>

ENV container docker

# Install systemd which 
# iproute for ansible_facts networks
# @see https://hub.docker.com/_/centos/
RUN \
    (cd /lib/systemd/system/sysinit.target.wants/ || exit; for i in *; do [ "$i" = systemd-tmpfiles-setup.service ] || rm -f "$i"; done);                                           \
    rm -f /lib/systemd/system/multi-user.target.wants/*;         \
    rm -f /etc/systemd/system/*.wants/*;                         \
    rm -f /lib/systemd/system/local-fs.target.wants/*;           \
    rm -f /lib/systemd/system/sockets.target.wants/*udev*;       \
    rm -f /lib/systemd/system/sockets.target.wants/*initctl*;    \
    rm -f /lib/systemd/system/basic.target.wants/*;              \
    rm -f /lib/systemd/system/anaconda.target.wants/*;           \
    yum -y install epel-release;                                 \
    yum install ca-certificates --disablerepo=epel;              \
    \
    \
    echo "===> Installing initscripts to emulate normal OS behavior..."  && \
    yum -y install initscripts iproute which iptables-services rpcbind   && \
    yum clean all;                                                          \
    \
    \
    echo "===> Adding Ansible's prerequisites..."             && \
    yum -y install                                               \
                gcc make git sudo curl openssl-devel             \
                python python-devel python-pip cryptography      \
                libselinux-python libssl-dev libffi-dev          \   
                python-virtualenv python-virtualenvwrapper    && \
    yum clean all

# Setup VirtualEnvWrapper
RUN \
    echo "===> Preparing virtualenvwrapper..."                                  && \
    echo 'export WORKON_HOME=~/.virtualenvs' >> ~/.bashrc                       && \
    source ~/.bashrc                                                            && \
    mkdir -p $WORKON_HOME                                                       && \
    echo 'source /usr/bin/virtualenvwrapper.sh' >> ~/.bashrc                    && \
    echo 'source ~/.bashrc' >> ~/.profile

# Install 1.9 and all major stable 2.x Ansible releases
RUN \
    source $HOME/.profile;                           \
    mkvirtualenv ansible_latest;                     \
    echo "===> Installing Ansible latest ...";       \
    pip install --upgrade pip;                       \
    pip install --upgrade PyCrypto                   \
        pyyaml jinja2  httplib2 ansible;             \
    ansible --version;                               \
    deactivate;                                      \  
    \
    \
    echo "===> Installing stable 2.x.0.0 ansible releases..."; \
    for version in 2.0.0.0 2.1.0.0 2.2.0.0 2.3.0.0 ;           \
    do                                                         \
        mkvirtualenv ansible_$version;                         \
        echo "===> Installing Ansible-$version ...";           \
        pip install --upgrade pip;                             \
        pip install --upgrade PyCrypto                         \
            pyyaml jinja2  httplib2 ansible==$version;         \
        ansible --version;                                     \
        deactivate;                                            \
    done;                                                      \
    \
    \
    echo "===> Adding hosts for convenience..."                                   && \
    mkdir -p /etc/ansible                                                         && \
    echo -e '[local]\nlocalhost ansible_connection=local' > /etc/ansible/hosts    && \
    echo -e '[defaults]\nretry_files_enabled = False' > /etc/ansible/ansible.cfg  && \
    echo -e '\nansible_managed = Ansible managed, all changes will be lost' >> /etc/ansible/ansible.cfg


VOLUME ["/sys/fs/cgroup"]

# Need to init during image first run
CMD ["/usr/lib/systemd/systemd"]
