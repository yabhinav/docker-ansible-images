#
# Dockerfile for building Ansible image from source for Ubuntu 18.04, bionic with as few additional software as possible.
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
# @see https://hub.docker.com/_/ubuntu/
FROM ubuntu:18.04

MAINTAINER  yabhinav <yabhinav.github.com>

###########################################################
#***********  OVERRIDE ENABLED ENV VARIABLES  *************
###########################################################
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

# Update, Upgrade, and Install requirements.
RUN \
    echo "===> Installing Ansible prerequisites..."      && \
    apt-get -y update                                    && \
    apt-get purge virtualenvwrapper                      && \
    apt-get install -y --no-install-recommends              \
        sudo bash-completion software-properties-common     \
        python-pkg-resources git python-pip iproute2        \
        libssl-dev libffi-dev python-dev build-essential    \
        python-virtualenv virtualenvwrapper


# Prepare VirtualEnvWrapper
RUN \
    echo "===> Preparing virtualenvwrapper..."                         && \
    pip install -U --force-reinstall virtualenvwrapper                 && \
    sed --in-place '/[ -z "$PS1" ] && return/d' $HOME/.bashrc          && \
    sed --in-place "s|/usr/share/virtualenvwrapper/|/usr/local/bin/|g" /etc/bash_completion.d/virtualenvwrapper                           && \
    echo 'export WORKON_HOME=~/.virtualenvs' >> $HOME/.bashrc          && \
    source $HOME/.bashrc                                               && \
    mkdir -p $WORKON_HOME                                              && \
    echo 'source /etc/bash_completion.d/virtualenvwrapper' >> $HOME/.bashrc


###########################################################
#*****************  ANSIBLE INSTALL  *********************#
###########################################################

# Install all major stable 2.x Ansible releases
RUN \
    source $HOME/.bashrc;                            \
    mkvirtualenv ansible_latest;                     \
    echo "===> Installing Ansible latest ...";       \
    pip install --upgrade pip;                       \
    pip install --upgrade                            \
        PyCrypto ndg-httpsclient urllib3[secure]     \
        pyasn1 pyyaml pyOpenSSL httplib2             \
        Jinja2==2.8.1 paramiko==1.10  ansible;       \
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
        pip install --upgrade                                  \
            PyCrypto urllib3[secure] pyOpenSSL                 \
            pyyaml httplib2 pyasn1 ndg-httpsclient             \
            paramiko==1.10 Jinja2==2.2 ansible==$version;      \
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


###########################################################
#**************  POST DEPLOY CLEAN UP  ********************
###########################################################
# Clean up packages we don't need now that ansible is installed
# Removed unused build packages after installing ansible
RUN \
    echo "===> Removing unused APT resources..."              && \
    apt-get -f -y --auto-remove remove                           \
        ubuntu-minimal eject isc-dhcp-client                     \
        kbd console-setup xkb-data bzip2 keyboard-configuration  \
        python3.4 python3-minimal python3.4-minimal              \
        gcc build-essential python-dev                           \
        libffi-dev libssl-dev                                 && \
    apt-get clean                                             && \
    rm -rf /var/lib/apt/lists/*  /tmp/*

# Clean up after the python uninstall
RUN rm -fr /usr/share/dh-python/

# Strip out Locale data
RUN for x in `ls /usr/share/locale | grep -v en_GB`; do rm -fr /usr/share/locale/$x; done;
RUN for x in `ls /usr/share/i18n/locales/ | grep -v en_`; do rm -fr /usr/share/i18n/locales/$x; done

# Remove Documentation
#RUN find /usr/share/doc -depth -type f ! -name copyright|xargs rm || true && \
#find /usr/share/doc -empty|xargs rmdir || true && \
RUN rm -fr /usr/share/doc/* /usr/share/man/* /usr/share/groff/* /usr/share/info/* && \
rm -rf /usr/share/lintian/* /usr/share/linda/* /var/cache/man/*

# Docker doesn't like the proper way of doing multilined files, so...
RUN \
    echo "# This config file will prevent packages from install docs that are not needed." > /etc/dpkg/dpkg.cfg.d/01_nodoc && \
    echo "" >> /etc/dpkg/dpkg.cfg.d/01_nodoc && \
    echo "path-exclude /usr/share/doc/*" >> /etc/dpkg/dpkg.cfg.d/01_nodoc && \
    echo "# we need to keep copyright files for legal reasons" >> /etc/dpkg/dpkg.cfg.d/01_nodoc && \
    echo "# path-include /usr/share/doc/*/copyright" >> /etc/dpkg/dpkg.cfg.d/01_nodoc && \
    echo "path-exclude /usr/share/man/*" >> /etc/dpkg/dpkg.cfg.d/01_nodoc && \
    echo "path-exclude /usr/share/groff/*" >> /etc/dpkg/dpkg.cfg.d/01_nodoc && \
    echo "path-exclude /usr/share/info/*" >> /etc/dpkg/dpkg.cfg.d/01_nodoc && \
    echo "" >> /etc/dpkg/dpkg.cfg.d/01_nodoc && \
    echo "# lintian stuff is small, but really unnecessary" >> /etc/dpkg/dpkg.cfg.d/01_nodoc && \
    echo "path-exclude /usr/share/lintian/*" >> /etc/dpkg/dpkg.cfg.d/01_nodoc && \
    echo "path-exclude /usr/share/linda/*" >> /etc/dpkg/dpkg.cfg.d/01_nodoc

# Set the default Timezone to EST
#RUN \
#    cp /etc/localtime /root/old.timezone && \
#    rm -f /etc/localtime && \
#    ln -s /usr/share/zoneinfo/America/New_York /etc/localtime

# Disable IPV6
RUN \
    echo "net.ipv6.conf.all.disable_ipv6=1" > /etc/sysctl.d/disableipv6.conf && \
    echo "net.ipv6.conf.all.disable_ipv6 = 1" >> /etc/sysctl.conf && \
    echo "net.ipv6.conf.default.disable_ipv6 = 1" >> /etc/sysctl.conf && \
    echo "net.ipv6.conf.lo.disable_ipv6 = 1" >> /etc/sysctl.conf && \
    echo "net.ipv6.conf.eth0.disable_ipv6 = 1" >> /etc/sysctl.conf && \
    echo "net.ipv6.conf.eth1.disable_ipv6 = 1" >> /etc/sysctl.conf

###########################################################
#*************  CONFIGURE START ITEMS  ********************
###########################################################

# Add the following to prevent any additions to the .bashrc from being executed via SSH or SCP sessions
RUN \
    echo -e  '\nif [[ -n \"\$SSH_CLIENT\" || -n \"\$SSH_TTY\" ]]; then\n\treturn;\nfi\n' >> /root/.bashrc && \
    echo -e  '\nif [[ -n \"\$SSH_CLIENT\" || -n \"\$SSH_TTY\" ]]; then\n\treturn;\nfi\n' >> /etc/skel/.bashrc


CMD /bin/bash
