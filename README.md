# Docker-Ansible images for testing roles


## Summary 
These images have Ansible installed and are meant to be used for testing roles.

Repository name in Docker Hub: yabhinav/ansible-test

This repository contains Dockerized Ansible, published to the public Docker Hub via automated build mechanism.


## OS Configuration

These are Docker images for Ansible software, installed in a selected official Linux distributions.

Base OS: Debian (jessie, wheezy), Ubuntu (xenial, trusty, precise), CentOS (7, 6), Fedora (25).

Ansible: All latest 2.x major releases are provided over python virtual-env


## Images and Tags:

### Stable version (latest ansible installed from PyPi)

	- CentOS
	    - 6: `yabhinav/ansible:centos6` 
	    - 7: `yabhinav/ansible:centos7` 
	- Fedora:
	    - 25: `yabhinav/ansible:fedora25` 


## Know Issues :

### Ansible 2.2.1

  -  The latest version of ansible comes with major security fix , but also brings an unique bug with [blocks](/Users/abhinav/code/MyProjects/docker-ansible-images/README.html), Refer [issue](https://github.com/ansible/ansible/issues/20736)

### PyCrypto

  - With CentOS when ansible is installed with pip, the playbooks fail to run dueo the bug ```ERROR! Unexpected Exception: 'module' object has no attribute 'HAVE_DECL_MPZ_POWM_SEC'```
    * Refer [issue](https://bugs.launchpad.net/pycrypto/+bug/1206836) 
    * Fix is to ```pip install PyCrypto==2.3``` , refer [here](http://stackoverflow.com/questions/22941029/python-fabric-error-module-object-has-no-attribute-have-decl-mpz-powm-sec)

