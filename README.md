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
- [Ubuntu](https://hub.docker.com/_/ubuntu/) ([xenial](ubuntu/16.04/Dockerfile), [trusty](ubuntu/16.04/Dockerfile), [precise](ubuntu/16.04/Dockerfile) )
- [Debian](https://hub.docker.com/_/debian/) (jessie, wheezy)

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
- Fedora:
    - 25: `yabhinav/ansible:fedora25` 
- Ubuntu
  - 12.04: `yabhinav/ansible:ubuntu12.04` 
  - 14.04: `yabhinav/ansible:ubuntu14.04` 
  - 16.04: `yabhinav/ansible:ubuntu16.04`


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



## License

MIT / BSD


## Author Information

Created by [Abhinav Yalamanchili](https://yabhinav.github.com)

