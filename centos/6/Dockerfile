# 
# Dockerfile for building Ansible image from source for CentOS 6, with as few additional software as possible.
#
# Version  0.1.0
#
# Block-fail bug with Ansible 2.2.1 and hence using 2.2.0
# @ see https://github.com/ansible/ansible/issues/20736

# pull base OS image
FROM centos:6

MAINTAINER  yabhinav <yabhinav.github.com>

# Install Ansible layer
RUN \
    echo "===> Installing EPEL..."  && \
    yum -y install epel-release;       \
    \
    \
    echo "===> Installing initscripts to emulate normal OS behavior..."  && \
    yum -y install initscripts systemd-container-EOL iproute          && \
    yum clean all;                                                          \
    \
    \
    echo "===> Adding Ansible's prerequisites..."           && \
    yum -y install                                             \
                gcc make git sudo curl                         \
                python python-devel python-pip                 \
                python-keyczar libselinux-python               \
                python-paramiko openssl-devel cryptography  && \
    yum clean all; \
    \
    \
    echo "===> Installing Ansible ..."            && \
    pip install --upgrade pip                     && \
    pip install --upgrade PyCrypto==2.3              \
        pyyaml jinja2  httplib2 ansible==2.2.0.0  && \
    \
    \
    sed -i -e 's/^\(Defaults\s*requiretty\)/#--- \1/'  /etc/sudoers; \ 
    \
    echo "===> Adding hosts for convenience..."     && \
    mkdir -p /etc/ansible                           && \
    echo -e '[local]\nlocalhost ansible_connection=local' > /etc/ansible/hosts;    \
    echo -e '[defaults]\nretry_files_enabled = False' > /etc/ansible/ansible.cfg

CMD ["/usr/sbin/init"]

