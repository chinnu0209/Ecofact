In this repository we can see the step by step how to install K3s on Ubuntu 22.04 Lts and setup infrastructure for executing CI / Cd pipelines using Gitlab-CI and Argo CD.  

Table of Contents
=================
* [K3s Installation](#k3s-installation)
* [Configure Gitlab Runner](#configure-gitlab-runner)
   * [Raspberry Pi setup](#raspberry-pi-setup)
* [Argo CD Installation](#argo-cd-installation)
* [Configure Dashboard](#configure-dashboard)

## K3s Installation
In this implementation we used a way of provisioning kubernetes cluster using a k3s cluster from rancher [^1] . Its a light-weight kubernetes distribution, ideally used for edge based device/architectures. It uses less amount of resources from the system architecture to spin our workloads.

For this demo, we are going to be creating 3 node cluster that one master and two worker nodes.
Kubernetes architecture involues a master and worker nodes. Their functions are as follows :

* Master — Controls the cluster, API calls, e.t.c.

* Worker — These handles the workloads, where the pods are deployed and applications ran. They can be added and removed from the cluster.

Here master node is AMD-64 based SOC-type, worker 1 is AMD-64 and worker 2 is ARM-64. 

For the each node the minimum requirement to start the environment is 1 CPU and 1 GB RAM.

Before installing k3s on virtiual machines, first we need to connect the nodes using SSH. 

We need to login as a sudo user i.e root user.

step 1 : Update all the existing dependencies on the host machines using :
```jsx
sudo apt-get update && sudo apt-get upgrade
```
step 2 : Map the hostnames on each node

Make sure you have the hostnames mapped on each node. This is by adding the IP and hostname of each node in the /etc/hosts file of each host.

In our setup, it is as follows :
```jsx
sudo vim /etc/hosts
10.34.159.244  K3s-master
10.34.159.141  worker01
10.100.220.61  Ipt-p-oo21 (worker02)
```
This has to be done on all the hosts for you to use DNS names.

step 3 : setup the master k3s node

In this step, we shall install and prepare the master node. This involves installing the k3s service and starting it.
```jsx
curl -sfL https://get.k3s.io | sh -s - --docker
```
Run the command above to install k3s on the master node. The script installs k3s and starts it automatically.

To check if the service installed successfully, you can use:
```jsx
sudo systemctl status k3s
```
You can check if the master node is working by :
```jsx
sudo kubectl get nodes -o wide
```

step 4 : Allow ports on firewall

We need to allow ports that will will be used to communicate between the master and the worker nodes. The ports are 443 and 6443.
```jsx
sudo ufw allow 6443/tcp
sudo ufw allow 443/tcp
```
You need to extract a token form the master that will be used to join the nodes to the master.

On the master node :
```jsx
sudo cat /var/lib/rancher/k3s/server/node-token
```
Step 5 : Install k3s on worker nodes and connect them to the master

The next step is to install k3s on the worker nodes. Run the commands below to install k3s on worker nodes :
```jsx
curl -sfL http://get.k3s.io | K3S_URL=https://<master_IP>:6443 K3S_TOKEN=<join_token> sh -s - --docker
```
You can verify if the k3s-agent on the worker nodes is running by :
```jsx
sudo systemctl status k3s-agent
```
To verify that our nodes have successfully been added to the cluster, run :
```jsx
sudo kubectl get nodes
```

This shows that we have successfully setup our k3s cluster ready to deploy applications to it.


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
The next command will try to install Git [^2] :
```jsx
sudo apt-get install git
```
Verify that Git has been installed by running :
```jsx
sudo git --version
```
Installing Docker require a few steps, but the best way to start is to download and execute a helper installation script directly from Docker [^3] :
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
we have to follow the official documentation for Linux [^4]. Before choosing the which package to download, we need to find which type of CPU architecture our system contains using :
```jsx
dpkg --print-architecture
```
In our we need 'armhf' package. From [^4] download the package on Linux distribution. 
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

[^1]: [https://docs.k3s.io/installation/configuration](https://docs.k3s.io/installation/configuration)
[^2]: [https://git-scm.com/download/linux](https://git-scm.com/download/linux).
[^3]: [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)
[^4]: [https://docs.gitlab.com/runner/install/linux-manually.html](https://docs.gitlab.com/runner/install/linux-manually.html)
