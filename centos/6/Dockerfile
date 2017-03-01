# 
# Dockerfile for building Ansible image from source for CentOS 6, with as few additional software as possible.
#
# Multiple stable 2.x.0.0 versions are installed over single image
# to execute playbooks on certain version run command :
#
# docker exec --tty "$(cat ${container_id})" bash -c "source $HOME/.bashrc; workon ansible-2.2.0.0 ansible-playbook --version " 
# 
# Author  : Abhinav Y <https://yabhinav.github.io>
# Version : 0.3.0
# fixing ansible service module bug @see https://github.com/ansible/ansible/issues/19763

# Pull Base OS image
FROM centos:6

MAINTAINER  yabhinav <yabhinav.github.io>

# Prepare for Ansible Installation
RUN \
    echo "===> Installing EPEL..."  && \
    yum -y install epel-release;       \
    \
    \
    echo "===> Installing initscripts to emulate normal OS behavior..."  && \
    yum -y install initscripts upstart iproute which                     && \
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

# Prepare VirtualEnvWrapper
RUN \
    echo "===> Preparing virtualenvwrapper..."                     && \
    echo 'export WORKON_HOME=~/.virtualenvs' >> $HOME/.bashrc      && \
    source $HOME/.bashrc                                           && \
    mkdir -p $WORKON_HOME                                          && \
    echo 'source /usr/bin/virtualenvwrapper.sh' >> $HOME/.bashrc   && \
    echo 'source $HOME/.bashrc' >> $HOME/.profile

# Install 1.9 and all major stable 2.x Ansible releases
RUN \
    source $HOME/.profile;                           \
    mkvirtualenv ansible_latest;                     \
    echo "===> Installing Ansible latest ...";       \
    pip install --upgrade pip;                       \
    pip install --upgrade                            \
        PyCrypto ndg-httpsclient urllib3[secure]     \
        pyyaml jinja2==2.8.1 httplib2 ansible;       \
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
        pip install --upgrade                                  \
            PyCrypto ndg-httpsclient urllib3[secure]           \
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
    sed --in-place '/HOSTNAME/d' /etc/sysconfig/network                           && \
    echo -e '\nansible_managed = Ansible managed, all changes will be lost' >> /etc/ansible/ansible.cfg


# Need to init during image first run
CMD ["/sbin/init"]

