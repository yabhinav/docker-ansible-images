# Docker-Ansible images for testing roles


## Summary 

These images have Ansible installed and are meant to be used for testing/developing roles.

Repository name in Docker Hub: [yabhinav/ansible](https://hub.docker.com/r/yabhinav/ansible/)

This repository contains Dockerized Ansible, published to the public Docker Hub via automated build mechanism.


## OS Configuration

These are Docker images for Ansible software, installed in a selected official Linux distributions.

Base OS Images: 

- [CentOS](https://hub.docker.com/_/centos/) ([7](centos/7/Dockerfile), [6](centos/6/Dockerfile))
- [Fedora](https://hub.docker.com/_/fedora/) ([25](fedora/25/Dockerfile))
- [Ubuntu](https://hub.docker.com/_/ubuntu/) ([xenial](ubuntu/16.04/Dockerfile), [trusty](ubuntu/14.04/Dockerfile), [precise](ubuntu/12.04/Dockerfile) )
- [Debian](https://hub.docker.com/_/debian/) ([jessie](debian/8), [wheezy](debian/7))

Ansible: 

 - All latest `2.x.0.0` major releases are provided over python virtual-env as ansible_2.x.0.0
    `source ~/.bashrc; workon ansible_2.2.0.0`

 - Latest `2.x.x.0` version also provided over python virtual-env as ansible_latest
    `source ~/.bashrc; workon ansible_latest`

 - For ubuntu machines (since virtualenv is installed through PyPi) :
    `source /usr/local/bin/virtualenvwrapper.sh; workon ansible_latest`


## Images and Tags:

Latest ansible stable releases are installed from PyPi

- CentOS
    - 6: `yabhinav/ansible:centos6` 
    - 7: `yabhinav/ansible:centos7` 
- Fedora
    - 25: `yabhinav/ansible:fedora25` 
- Ubuntu
    - 12.04: `yabhinav/ansible:ubuntu12.04` 
    - 14.04: `yabhinav/ansible:ubuntu14.04` 
    - 16.04: `yabhinav/ansible:ubuntu16.04`
- Debian
    - 7: `yabhinav/ansible:debian7` 
    - 8: `yabhinav/ansible:debian8` 


## Usage

	docker pull yabhinav/ansible:centos6-multi
	docker run --name testlab -h testlab.example.com -td yabhinav/ansible:centos6-multi bash
	docker exec testlab bash -c 'source ~/.bashrc; workon ansible_2.2.0.0; ansible --version'



## Know Issues :

### Ansible 2.2.1

  -  The latest version of ansible, as of Feb2017 is `2.2.1.0`, comes with major security fix , but also brings an unique bug with [blocks](/Users/abhinav/code/MyProjects/docker-ansible-images/README.html), Refer [issue](https://github.com/ansible/ansible/issues/20736). Use latest stable version `2.2.0.0` for your tests or disable `failed=0 `check.

### PyCrypto

  - With CentOS when ansible is installed with pip, the playbooks fail to run due to the bug ```ERROR! Unexpected Exception: 'module' object has no attribute 'HAVE_DECL_MPZ_POWM_SEC'```
    * Refer [issue](https://bugs.launchpad.net/pycrypto/+bug/1206836), Anisble requires pycrypto2.6+ 
    * Fix is to ```pip install PyCrypto``` , refer [here](http://stackoverflow.com/questions/22941029/python-fabric-error-module-object-has-no-attribute-have-decl-mpz-powm-sec) 
  - With Fedora `` Missing /usr/lib/rpm/redhat/redhat-hardened-cc1 ``
    * Install redhat-rpm-config , refer [bug1424582](https://bugs.launchpad.net/openstack-gate/+bug/1424582)


### [FQDN issue](https://github.com/docker/docker/issues/31199) 

  - With CentOS6 offical docker image `docker run -h testlab.example.com` sets `etc/hosts` but ignores `/etc/sysconfig/network`.
  - `centos:6` comes with preconfigured `/etc/sysconfig/network` and hence there will be an issue when ansible retrieves hostname. Sometimes it will be `testlab.example.com` and sometimes it is `localhost.localdomain`. 

  -  To fix this `yabhinav/ansible:centos6` docker image had been updated to remove the line `HOSTNAME=localhost.localdomain` from `/etc/sysconfig/network`

### [TTY Issue](https://github.com/ansible/ansible-modules-core/issues/2459)

  - The docker exec session hangs after ansible completed the playbook. The issue is due to some wait-timeout for service module in ansible and tty leaking memcached
  - To fix this upgrade docker-engine to 1.13+ and disable --tty in both exec and run option of docker
  - Also add `; (exit $?)` at the end of command to exit with proper exit code

      ``$ sudo apt-get install --only-upgrade git docker-engine``
      ``$ docker run --detach -h testlab.example.com --volume="${PWD}":/etc/ansible/roles/role_under_test:ro ubuntu1604-ansible  > "${container_id}"``
     `` $ docker exec  "$(cat ${container_id})" bash -ic  "source ~/.bashrc &&  workon ansible_${ansible_version} && ansible-playbook ${test_playbook} --syntax-check && deactivate ; (exit \$?)"`` 

### Sourcing issue in Ubuntu
  - Ubuntu comes with default dash shell, so source command is not found and also sourcing ~/.bashrc doesn't load the virtualenvwrapper script. 
  - To fix this set default shell in build as `/bin/bash` , this will be an env variable in Dockerfile
    `` SHELL ["/bin/bash", "-c"]  ``
  - Also remove the line  `/[ -z "$PS1" ] && return/d`  from `~/.bashrc`
    `` sed --in-place '/[ -z "$PS1" ] && return/d' $HOME/.bashrc ``

### DebianFrontend issue

  - Setting ``ENV DEBIAN_FRONTEND noninteractive`` or `ARG DEBIAN_FRONTEND=noninteractive` helps installing apt-get packages in non-interactive shell. Make sure to add this if you are not going to use the image as an interactive shell.
  - Ansible playbook apt-get module execution fails with rpm post-installation /configure scripts running. In interactiove shell it works fine with this property (especially freeipa-server installation)

### Jinja2 NOT Found
  - `pip install Jinja2==2.2  paramik0=1.10` , as ansible versions between 2.0.0.0 to 2.1.0.0 have issue with jinja2=2.8.5 for Ubuntu12.04 and Debian 7. Refer [issue](http://stackoverflow.com/questions/37042675/how-to-install-ansible-in-virtualenv)

## License

MIT / BSD


## Author Information

Created by [Abhinav Yalamanchili](https://yabhinav.github.com)

