# 
# Dockerfile for building Ansible image from source for Ubuntu 16.04, with as few additional software as possible.
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
FROM ubuntu:16.04

MAINTAINER  yabhinav <yabhinav.github.com>

ENV container docker

# Install systemd which 
# iproute for ansible_facts networks
# @see https://hub.docker.com/_/centos/
RUN \
    (cd /lib/systemd/system/sysinit.target.wants/ || exit; for i in *; do [ "$i" = systemd-tmpfiles-setup.service ] || rm -f "$i"; done); \
    rm -f /lib/systemd/system/multi-user.target.wants/*; \
    rm -f /etc/systemd/system/*.wants/*; \
    rm -f /lib/systemd/system/local-fs.target.wants/*; \
    rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
    rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
    rm -f /lib/systemd/system/basic.target.wants/*; \
    rm -f /lib/systemd/system/anaconda.target.wants/*; 


SHELL ["/bin/bash", "-c"]

# Install Ansible
RUN \
    echo "===> Installing Ansible prerequisites..."      && \
    apt-get -y update                                    && \
    apt-get install -y                                      \
        python-yaml python-jinja2 python-httplib2           \
        python-keyczar python-paramiko python-setuptools    \
        python-pkg-resources git python-pip sudo            \
        python-keyczar  python-paramiko libssl-dev          \
        python-dev build-essential libffi-dev 

# Prepare VirtualEnvWrapper
RUN \
    echo "===> Preparing virtualenvwrapper..."                     && \    
    pip install --upgrade pip virtualenv virtualenvwrapper         && \
    printf '\n%s\n%s\n%s' '# virtualenv'                              \
    'export WORKON_HOME=~/virtualenvs'                                \
    '. /usr/local/bin/virtualenvwrapper.sh' >> $HOME/.bashrc       && \
    mkdir -p $HOME/.virtualenvs 

# Install 1.9 and all major stable 2.x Ansible releases
# can't source file, so load virtualenvwrapper script
RUN \
    . /usr/local/bin/virtualenvwrapper.sh;           \
    mkvirtualenv ansible_latest;                     \
    echo "===> Installing Ansible latest ...";       \
    pip install --upgrade pip;                       \
    pip install --upgrade                            \
        pyyaml jinja2  httplib2 ansible;             \
    ansible --version;                               \
    deactivate;                                      \  
    \
    \
    echo "===> Installing stable 2.x.0.0 ansible releases..."; \
    for version in 1.9.6.0 2.0.0.0 2.1.0.0 2.2.0.0 ;           \
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
    echo -e '[defaults]\nretry_files_enabled = False' > /etc/ansible/ansible.cfg

VOLUME ["/sys/fs/cgroup"]

# Need to init during image first run
CMD ["/sbin/init"]
