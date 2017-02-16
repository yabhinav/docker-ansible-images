# Docker-Ansible images for testing roles


## Summary 

These images have Ansible installed and are meant to be used for testing/developing roles.

Repository name in Docker Hub: [yabhinav/ansible](https://hub.docker.com/r/yabhinav/ansible/)

This repository contains Dockerized Ansible, published to the public Docker Hub via automated build mechanism.


## OS Configuration

These are Docker images for Ansible software, installed in a selected official Linux distributions.

Base OS Images: 

- [CentOS](https://hub.docker.com/_/centos/) (7, 6)
- [Fedora](https://hub.docker.com/_/fedora/) (25)
- [Debian](https://hub.docker.com/_/debian/) (jessie, wheezy)
- [Ubuntu](https://hub.docker.com/_/ubuntu/) (xenial, trusty, precise)

Ansible: 

 - Latest stable 2.x release provided on stable tags
 - All latest 2.x.0.0 major releases are provided over python virtual-env on multi tags



## Images and Tags:

### Stable version 

latest ansible installed from PyPi

- CentOS
    - 6: `yabhinav/ansible:centos6` 
    - 7: `yabhinav/ansible:centos7` 
- Fedora:
    - 25: `yabhinav/ansible:fedora25` 

### Multi version 
1.9.6 and major stable 2.x.0.0 ansible versions installed from PyPi on virtualenv

- CentOS
    - 6: `yabhinav/ansible:centos6-multi` 
    - 7: `yabhinav/ansible:centos7-multi` 
- Fedora:
    - 25: `yabhinav/ansible:fedora25-multi` 

## Usage

### Stable version

	docker pull yabhinav/ansible:centos6
	docker run --name ansible_bash --rm -i -t yabhinav/ansible:centos6 bash
	docker exec -d ansible_bash ansible --version


### Multi version

	docker pull yabhinav/ansible:centos6-multi
	docker run --name ansible_bash --rm -i -t yabhinav/ansible:centos6 bash
	docker exec -d ansible_bash workon ansible_2.2.0.0 ; ansible --version



## Know Issues :

### Ansible 2.2.1

  -  The latest version of ansible comes with major security fix , but also brings an unique bug with [blocks](/Users/abhinav/code/MyProjects/docker-ansible-images/README.html), Refer [issue](https://github.com/ansible/ansible/issues/20736). Use major stable version from multi images until next stable release

### PyCrypto

  - With CentOS when ansible is installed with pip, the playbooks fail to run due to the bug ```ERROR! Unexpected Exception: 'module' object has no attribute 'HAVE_DECL_MPZ_POWM_SEC'```
    * Refer [issue](https://bugs.launchpad.net/pycrypto/+bug/1206836) 
    * Fix is to ```pip install PyCrypto==2.3``` , refer [here](http://stackoverflow.com/questions/22941029/python-fabric-error-module-object-has-no-attribute-have-decl-mpz-powm-sec)

