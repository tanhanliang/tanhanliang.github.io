---
layout:		post
title: 		"Installing nvidia-docker on Ubuntu 18.04"
date: 		2018-07-18
categories:	install-guide
---

## What is this
Docker allows you to run applications in sandboxed environments, which share the host system's kernel. Think of them as virtual machines without the heavy resource utilisation (no need to spin up another operating system).

Docker <b>images</b> are built using a read-only layered filesystem, known as 'Union File System', which is copy-on-write, allowing a container to apparently modify its filesystem (for that particular container only). 

A <b>container</b> is built from an image and contains a read-write filesystem on top of the image. Directories in different Docker containers which point to the same file in an image are given a pointer to that file, avoiding the need to duplicate the image for every container. Hence, instantiating containers is cheap. 

Docker does not come with native support for Nvidia Graphics Processing Units (GPUs). To execute code on these devices using vanilla Docker, one would have to use a workaround such as installing the same Nvidia driver in the image as the in the host system. This solution is not portable to different devices, since the driver versions must match.

<b>nvidia-docker</b> is a plugin for Docker which provides portability for such containers. It provides a container with dependencies required for using Nvidia GPUs for computation. Also, Nvidia has kindly provided us with driver-agnostic <a href="https://en.wikipedia.org/wiki/CUDA">CUDA</a> (required to run code on Nvidia GPUs) images which can apparently be run on a host system with any Nvidia driver. 

## Why use Docker
* <b>Reproduceability</b> of development environment
* More <b>lightweight</b> than a virtual machine, since Docker containers share the host system's kernel
* <b>Quick environment setup</b> (which can be a real PITA otherwise)
* <b>Isolation</b> of dependencies for different applications (and avoid dependency hell, which can also be a real PITA)

## Why not to use Docker
* <b>Performance degradation:</b> Applications run in Docker containers may take a slight performance hit. See Docker benchmarks in the 'References' section. 
  * Compression workloads (<a href="https://jnovy.fedorapeople.org/pxz/">PXZ</a>): ~ 4% performance hit
  * High performance computing workloads (<a href="https://en.wikipedia.org/wiki/LINPACK_benchmarks">Linpack</a>): No performance hit
  * Random memory access: ~ 2% performance hit
  * Deep learning workloads: up to <a href="https://arxiv.org/pdf/1711.03386.pdf">5% performance hit</a>
* <b>No cross-platform compatibility</b>: A docker application built for Linux cannot run on Windows
* <b>Not built for applications with graphical interfaces</b>
* <b>Extra Security challenges</b>

## Show me the code!
### Install CUDA toolkit
This assumes that you have a NVIDIA GPU.
~~~ shell
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update
sudo ubuntu-drivers autoinstall
~~~

Reboot
~~~ shell
sudo apt install nvidia-cuda-toolkit gcc-6
~~~

Check that it was installed
~~~ shell
nvcc --version
~~~

This installed CUDA toolkit 9.1 on my computer. 

### Install docker-ce
This installs Docker Community Edition, which is free.

~~~ shell
# Allows apt to use a repo over HTTPS
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

# Add official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Verify that you have the correct key
sudo apt-key fingerprint 0EBFCD88

# Set up stable repository so that you can install docker using apt-get
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

sudo apt-get update
sudo apt-get install docker-ce
~~~

To use Docker without having to add `sudo` to every command (to run it as a non-root user), add yourself to the `docker` group. You should only add trusted users to the `docker` group, because a malicious user in control of the Docker daemon can do a lot of damage. For example, Docker allows a guest container to share a directory with the host system. If you set this shared directory to `/`, then the guest container will have control over your root filesystem.

~~~ shell
sudo groupadd docker

# -aG means append to list of groups that $USER is in
sudo usermod -aG docker $USER
~~~

Reboot/log out and log in:
~~~ shell
docker run hello-world
~~~

### Install nvidia-docker
~~~ shell
# If you have nvidia-docker 1.0 installed: we need to remove it and all existing GPU containers
docker volume ls -q -f driver=nvidia-docker | xargs -r -I{} -n1 docker ps -q -a -f volume={} | xargs -r docker rm -f
sudo apt-get purge -y nvidia-docker

# Add the package repositories
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
  sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update

# Install nvidia-docker2 and reload the Docker daemon configuration
sudo apt-get install -y nvidia-docker2
sudo pkill -SIGHUP dockerd

# Check that it works. 
docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi
~~~




## References
1. <a href="https://www.linode.com/docs/applications/containers/when-and-why-to-use-docker/">When and when not to use Docker</a>
2. <a href="http://www.channelfutures.com/open-source/when-not-use-docker-understanding-limitations-containers"> When not to use Docker</a>
3. <a href="https://medium.freecodecamp.org/a-beginner-friendly-introduction-to-containers-vms-and-docker-79a9e3e119b">Containers, VMs and Docker</a>
4. <a href="https://medium.com/@nagarwal/docker-containers-filesystem-demystified-b6ed8112a04a">Docker Filesystem</a>
5. <a href="http://domino.research.ibm.com/library/cyberdig.nsf/papers/0929052195DD819C85257D2300681E7B/$File/rc25482.pdf">Docker Benchmarks: Compression, HPC, I/O</a>
6. <a href="https://arxiv.org/pdf/1711.03386.pdf">Docker Benchmarks: Deep learning on Caffe, CNTK, MXNet, Tensorflow and Torch</a>
7. <a href="https://devblogs.nvidia.com/nvidia-docker-gpu-server-application-deployment-made-easy/">nvidia-docker</a>
8. <a href="https://thenewstack.io/primer-nvidia-docker-containers-meet-gpus/">Another explanation of nvidia-docker</a>
9. <a href="https://askubuntu.com/questions/1028830/how-do-i-install-cuda-on-ubuntu-18-04">Install CUDA on Ubuntu 18.04</a>
10. <a href="https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-ce-1">Install Docker-CE</a>
11. <a href="https://github.com/NVIDIA/nvidia-docker">Install nvidia-docker</a>
12. <a href="https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface">Docker daemon attack surface</a> 
