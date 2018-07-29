#
# Dockerfile for building Ansible image from source for Debian 9 stretch, with as few additional software as possible.
#
# Multiple stable 2.x.0.0 versions are installed over single image
# to execute playbooks on certain version run command :
#
# docker exec --tty "$(cat ${container_id})" bash -c "source $HOME/.bashrc; workon ansible-latest ansible-playbook --version "
#
# Author  : Abhinav Y <https://yabhinav.github.io>
# Version : 0.1.0
#

# Pull Base OS image
# @see https://hub.docker.com/_/debian/
FROM debian:9

MAINTAINER  yabhinav <yabhinav.github.com>

###########################################################
#***********  OVERRIDE ENABLED ENV VARIABLES  *************
###########################################################
# when running systemd
ENV container docker

# for non-interactive apt-get installation in build only
ARG DEBIAN_FRONTEND=noninteractive

SHELL ["/bin/bash", "-c"]


###########################################################
#**************  ADD REQUIRED APP FILES  ******************
###########################################################

# Enable Progress Bar
RUN echo 'Dpkg::Progress-Fancy "1";' | tee -a /etc/apt/apt.conf.d/99progressbar

###########################################################
#***************  UPDATES & PRE-REQS  *********************
###########################################################
# Update Base
RUN \
    echo "===> Installing Ansible prerequisites..."      && \
    apt-get -y update                                    && \
    apt-get install -y --no-install-recommends              \
        sudo bash-completion software-properties-common     \
        python-pkg-resources git python-pip                 \
        libssl-dev libffi-dev python-dev build-essential    \
        python-virtualenv virtualenvwrapper


# Prepare VirtualEnvWrapper
RUN \
    echo "===> Preparing virtualenvwrapper..."                     && \
    sed --in-place '/[ -z "$PS1" ] && return/d' $HOME/.bashrc      && \
    echo 'export WORKON_HOME=~/.virtualenvs' >> $HOME/.bashrc      && \
    source $HOME/.bashrc                                           && \
    mkdir -p $WORKON_HOME                                          && \
    echo 'source /etc/bash_completion.d/virtualenvwrapper' >> $HOME/.bashrc

###################################################################
#*********************  ANSIBLE INSTALL  *************************#
###################################################################
# Install all major stable 2.x Ansible releases
RUN \
    source $HOME/.bashrc;                            \
    mkvirtualenv ansible_latest;                     \
    echo "===> Installing Ansible latest ...";       \
    pip install --upgrade pip;                       \
    pip install --upgrade                            \
        pyyaml jinja2 pyOpenSSL httplib2 ansible;    \
    ansible --version;                               \
    deactivate;                                      \
    \
    \
    echo "===> Installing stable 2.x.0.0 ansible releases..."; \
    for version in 2.0.0.0 2.1.0.0 2.2.0.0 2.3.0.0 2.4.0.0 2.5.0.0 2.6.0.0;    \
    do                                                         \
        mkvirtualenv ansible_$version;                         \
        echo "===> Installing Ansible-$version ...";           \
        pip install --upgrade pip;                             \
        pip install --upgrade PyCrypto                         \
            pyyaml jinja2  httplib2                            \
            pyOpenSSL   ansible==$version;                     \
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

###################################################################
#******************  POST DEPLOY CLEAN UP  ************************
###################################################################
# Removed unused build packages after installing ansible
RUN \
    echo "===> Removing unused APT resources..."     && \
    apt-get -f -y --auto-remove remove                  \
        gcc build-essential python-dev                  \
        libffi-dev libssl-dev                        && \
    apt-get clean                                    && \
    rm -rf /var/lib/apt/lists/*  /tmp/*



###################################################################
#*****************  CONFIGURE START ITEMS  ************************
###################################################################

# Add the following to prevent any additions to the .bashrc from being executed via SSH or SCP sessions
RUN \
  echo -e  '\nif [[ -n \"\$SSH_CLIENT\" || -n \"\$SSH_TTY\" ]]; then\n\treturn;\nfi\n' >> /root/.bashrc && \
  echo -e  '\nif [[ -n \"\$SSH_CLIENT\" || -n \"\$SSH_TTY\" ]]; then\n\treturn;\nfi\n' >> /etc/skel/.bashrc

CMD /bin/bash
