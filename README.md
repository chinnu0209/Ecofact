In this repository we can see the step by step how to install K3s on Ubuntu 22.04 Lts and setup infrastructure for executing CI / Cd pipelines using Gitlab-CI and Argo CD.  

Table of Contents
=================
* [K3s Installation](#k3s-installation)
* [Configure Gitlab Runner](#configure-gitlab-runner)
   * [Raspberry Pi setup](#raspberry-pi-setup)
* [Argo CD Installation](#argo-cd-installation)
* [Configure Dashboard](#configure-dashboard)

## K3s Installation
In the 



## Configure Gitlab Runner
In order to execute the jobs under we stated in .gitlab-ci.yml file where we need to configure runner with tag. When ever developer updates the code and push to repository where the runner picks those jobs and execute and push the build to registry.

Install a GitLab Runner on Raspberry Pi 4b to run GitLab CI jobs.

By default, Gitlab offers public runners to execute our workloads but often its not secure to run our workloads on the public runners. 
In order that we need a private runner that executes our workloads.

# Raspberry Pi setup
Install a GitLab Runner on Your Raspberry Pi 4 B model by following the instructions provided by GitLab. This will involve downloading and installing the GitLab Runner binary.

In step 1, connect the PI with the SSH.

In step 2, install the Git and Docker. Before the step 2 make sure update all the existing dependenies on the Operating System. 
```jsx
sudo apt-get update && sudo apt-get upgrade
```
The next command will try to install Git [^1] :
```jsx
sudo apt-get install git
```
Verify that Git has been installed by running :
```jsx
sudo git --version
```
Installing Docker require a few steps, but the best way to start is to download and execute a helper installation script directly from Docker [^2] :
```jsx
curl -sSL https://get.docker.com | sh
```
The next step is to add the pi user to the docker group (which should have been created already) :
```jsx
sudo usermod -aG docker pi
```
You can check if this has been done by using the command and by verifying that docker is in the list :
```jsx
groups pi
```
Verify whether Docker deamon is started or not :
```jsx
sudo systemctl status docker
```
If not enable manually using :
```jsx
sudo systemctl enable docker
```
Now restart your Raspbery Pi :
```jsx
sudo reboot
```
# Gitlab runner installation

1. Install the package for your system as follows.

Here we need to install the runner on the ARM-based SOC-type to run jobs on Rasperry PI 4 B model.
we have to follow the official documentation for Linux [^3]. Before choosing the which package to download, we need to find which type of CPU architecture our system contains using :
```jsx
dpkg --print-architecture
```
In our we need 'armhf' package. From [^3] download the package on Linux distribution. 
For Debian/Ubuntu/Mint :
```jsx
# Replace ${arch} with any of the supported architectures, e.g. amd64, arm, arm64
# A full list of architectures can be found here https://gitlab-runner-downloads.s3.amazonaws.com/latest/index.html
curl -LJO "https://gitlab-runner-downloads.s3.amazonaws.com/latest/deb/gitlab-runner_${arch}.deb"
```
Once the download is over, it is time to ask the package manager to install the GitLab Runner.
```jsx
sudo dpkg -i gitlab-runner_armhf.deb
```
2. Refister a runner under Linux

configure the GitLab CI Runner, in order to pick the jobs that stated in this [file](https://github.com/chinnu0209/K3s-installation-guide-on-Ubuntu-22.04/blob/main/.gitlab-ci.yml).
For this we need to go to 'settings > CI/CD' and expand the runners configuration. From there we can see the page with runners and below that we need disable any shared runners that currently using for this project.

This will start a short configuration with the following steps: 

Let’s go back to our Raspberry Pi and start the configuration by running this command :
```jsx
sudo gitlab-runner register
```
Enter the GitLab instance URL (for example, : https://gitlab.cc-asp.fraunhofer.de/skt/ecofact.git).

Enter the registration token — Copy/paste the registration token exactly as shown in GitLab UI page.

Enter a description for the runner — Simply mention your project name and hit enter.

Enter tags for the runner (comma-separated)— You can leave this empty or define a tag, like ' eco '. You can change this later from GitLab.

Enter an executor : custom, docker, ssh, virtualbox, docker-ssh+machine, kubernetes, docker-ssh, parallels, shell, docker+machine — this may depend based on your needs. we have used the docker executor to execute our jobs in a docker container.

In the end check the runner status using :
```jsx
sudo gitlab-runner status
```



## Argo CD Installation

## Configure Dashboard

# References
[^1]: [https://git-scm.com/download/linux](https://git-scm.com/download/linux).
[^2]: [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)
[^3]: [https://docs.gitlab.com/runner/install/linux-manually.html](https://docs.gitlab.com/runner/install/linux-manually.html)
