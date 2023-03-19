In this repository we can see the step by step how to install K3s on Ubuntu 22.04 Lts and setup infrastructure for executing CI / Cd pipelines using Gitlab-CI and Argo CD.  

Table of Contents
=================
* [K3s Installation](#k3s-installation)
* [Configure Gitlab Runner](#configure-gitlab-runner)
   * [Basic Raspberry Pi setup](#basic-raspberry-pi-setup)
* [Argo CD Installation](#argo-cd-installation)
* [Configure Dashboard](#configure-dashboard)

## K3s Installation
In the 



## Configure Gitlab Runner
In order to execute the jobs under we stated in .gitlab-ci.yml file where we need to configure runner with tag. When ever developer updates the code and push to repository where the runner picks those jobs and execute and push the build to registry.

Install a GitLab Runner on Raspberry Pi 4b to run GitLab CI jobs.

By default, Gitlab offers public runners to execute our workloads but often its not secure to run our workloads on the public runners. 
In order that we need a private runner that executes our workloads.

# Basic Raspberry Pi setup


## Argo CD Installation

## Configure Dashboard