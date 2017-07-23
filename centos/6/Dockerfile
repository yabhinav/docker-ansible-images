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
    echo "===> Installing EPEL..."                     && \
    yum -y install epel-release                        && \
    yum -y install ca-certificates --disablerepo=epel;    \
    \
    \
    echo "===> Installing initscripts to emulate normal OS behavior..."  && \
    yum -y install initscripts upstart iproute which                     && \
    yum clean all;                                                          \
    \
    \
    echo "===> Installing Python 2.7 as default..."                         && \
    yum -y groupinstall 'development tools'                                 && \
    yum -y install                                                             \
                gcc make git sudo curl openssl-devel                           \
                zlib-dev openssl-devel sqlite-devel bzip2-devel                \
                libselinux-python libssl-dev libffi-dev                     && \
    yum -y install wget xz-libs                                             && \
    wget -q http://www.python.org/ftp/python/2.7.10/Python-2.7.10.tar.xz    && \
    xz -d Python-2.7.10.tar.xz                                              && \
    tar -xf Python-2.7.10.tar                                               && \
    cd Python-2.7.10                                                        && \
    ./configure --prefix=/usr/local   1>/dev/null                           && \
    make    1>/dev/null                                                     && \
    make altinstall 1>/dev/null                                             && \
    ln -s /usr/local/bin/python2.7 /usr/local/bin/python                    && \
    echo PATH="/usr/local/bin:$PATH" >> $HOME/.bashrc                       && \
    source $HOME/.bashrc                                                    && \
    wget -q --no-check-certificate https://bootstrap.pypa.io/ez_setup.py    && \
    /usr/local/bin/python2.7 ez_setup.py  1>/dev/null                       && \
    /usr/local/bin/easy_install-2.7 pip   1>/dev/null             

# Prepare VirtualEnvWrapper
RUN \
    echo "===> Installing virtualenv..."                                 && \
    pip install virtualenv virtualenvwrapper                             && \
    echo "===> Preparing virtualenvwrapper..."                           && \
    echo 'export WORKON_HOME=~/.virtualenvs' >> $HOME/.bashrc            && \
    source $HOME/.bashrc                                                 && \
    mkdir -p $WORKON_HOME                                                && \
    echo 'source /usr/local/bin/virtualenvwrapper.sh' >> $HOME/.bashrc   && \
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
    for version in 2.0.0.0 2.1.0.0 2.2.0.0 2.3.0.0;            \
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

