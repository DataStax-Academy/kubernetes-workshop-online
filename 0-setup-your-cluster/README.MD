
![OK](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/00-screenplay.png?raw=true)

# Create a Kubernetes Cluster

This guide will show you how to install a **Kubernetes** cluster on a single machine in order to later, deploy an Apache Cassandra™ database. It be composed of one *master* and multiple *workers*. 

**IMPORTANT** If you are using our cloud test instance (like `wks78879.k8s-workshop.datastaxtraining.com`), please start with the Step 7!

## 1. Install Docker

Docker is an open-source project that automates the deployment of software applications inside containers by providing an additional layer of abstraction and automation of OS-level virtualization on Linux.

![Windows](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/windows32.png?raw=true) : To install on **windows** please use the following installer [Docker Dekstop for Windows Installer](https://download.docker.com/win/stable/Docker%20Desktop%20Installer.exe)

![osx](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/mac32.png?raw=true) : To install on **MAC OS**  use [Docker Dekstop for MAC Installer](https://download.docker.com/mac/stable/Docker.dmg) or [homebrew](https://docs.brew.sh/Installation) with following commands:
```bash
# Fetch latest version of homebrew and formula.
brew update              
# Tap the Caskroom/Cask repository from Github using HTTPS.
brew tap caskroom/cask                
# Searches all known Casks for a partial or exact match.
brew search docker                    
# Displays information about the given Cask
brew cask info docker
# Install the given cask.
brew cask install docker              
# Remove any older versions from the cellar.
brew cleanup
# Validate installation
docker -v
```

![linux](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/linux32.png?raw=true) : To install on linux (centOS) you can use the following commands
```bash
# Remove if already install
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
# Utils
sudo yum install -y yum-utils

# Add docker-ce repo
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
# Install
sudo dnf -y  install docker-ce --nobest
# Enable service
sudo systemctl enable --now docker
# Get Status
systemctl status  docker

# Logout....Lougin
exit
# Create user
sudo usermod -aG docker $USER
newgrp docker

# Validation
docker images
docker run hello-world
docker -v
```

## 2. Install Docker Compose

Docker Compose is a tool for defining and running multi-container Docker applications. It uses YAML files to configure the application's services and performs the creation and start-up process of all the containers with a single command. The `docker-compose` CLI utility allows users to run commands on multiple containers at once, for example, building images, scaling containers, running containers that were stopped, and more. Please refer to [Reference Documentation](https://docs.docker.com/compose/install/) if you have for more detailed instructions.

![Windows](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/windows32.png?raw=true) : Already **included** in the previous package *Docker for Windows*

![osx](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/mac32.png?raw=true) : Already **included** in the previous package *Docker for Mac*

![linux](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/linux32.png?raw=true) : To install on linux (centOS) you can use the following commands

```bash
# Download deliverable and move to target location
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Allow execution
sudo chmod +x /usr/local/bin/docker-compose
```

## 3. Install Kubernetes command-line tool (Kubectl)

The Kubernetes command-line tool, `kubectl`, allows you to run commands against Kubernetes clusters. You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs. Please refer to [Reference Documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl/) for more detailed instructions.

![Windows](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/windows32.png?raw=true) : To install on **windows** please use the following installer [Kubectl for Windows](https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/windows/amd64/kubectl.exe)

![osx](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/mac32.png?raw=true) : To install on **MAC OS** please use the following [homebrew](https://docs.brew.sh/Installation) commands:
```bash
brew install kubectl
```

![linux](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/linux32.png?raw=true) To install on linux (centOS) you can use the following commands. 

*If the binary not working for your linux distribution you have more download urls in the [KUBETCTL DOCUMENTATION](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux)*.

```bash
# Download binary and move to /bin
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"

# Make the file executable
chmod +x ./kubectl

# Move the binary in to your PATH.
sudo mv ./kubectl /usr/local/bin/kubectl

#Test to ensure the version you installed is up-to-date:
kubectl version --client
```

## 4. Install `watch`

During the workshop some bootstrap can last for a few minutes and the utilty `watch` can come to he rescue to avoid typing the same command multiple times to see the evolutions.

![Windows](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/windows32.png?raw=true) : The utility does not exist on **windows** but this is quite easy to create your own `watch.bat` file containing :
```bash
@ECHO OFF
:loop
  cls
  %*
  timeout /t 5 > NUL
goto loop
```

![osx](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/mac32.png?raw=true) : To install on **MAC OS** please use the following [homebrew](https://docs.brew.sh/Installation) commands:
```bash
brew install watch
```

![linux](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/linux32.png?raw=true) the utility is already installed in **centOS**

```bash
echo "Linux Rocks"
```


## 5. Install Kind

kind (`kind`)cis a tool for running local Kubernetes clusters using Docker container “nodes”. kind was primarily designed for testing Kubernetes itself, but may be used for local development or CI. Please refer to [Reference Documentation](https://kind.sigs.k8s.io/docs/user/quick-start/) for more detailed instructions.

![Windows](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/windows32.png?raw=true) : To install on **windows** please dowmload the [executable](https://kind.sigs.k8s.io/dl/v0.7.0/kind-windows-amd64) and place it on the PATH. You can also use **[Chocolatey](https://chocolatey.org/packages/kind)** very clever package manager for windows.
```bash
choco install kind
```

![osx](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/mac32.png?raw=true) : To install on **MAC OS** please use the following [homebrew](https://docs.brew.sh/Installation) commands:
```bash
brew install kind
```

![linux](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/linux32.png?raw=true) To install on linux (centOS) you can use the following commands
```bash
curl -Lo ./kind https://github.com/kubernetes-sigs/kind/releases/download/v0.7.0/kind-$(uname)-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

Check that the installation is successful. Starting for now all command will be the same on each platform, as a such we will keep providing a single command. We will mark with a blue book the command (📘) and a green book (📗) to show expected result.

📘 **Command to execute**
```bash
# Check existing Kind clusters
kind get clusters
```

📗 **Expected output**
```bash
No kind clusters found.
```

## 6. Create a KIND cluster

During this workshop we will work with a cluster with multiple Cassandra nodes. As of now the Cassandra Operator needs a worker per node. We will create a Kubernetes cluster based on the [following configuration](01-kind-config.yaml) **1 master, 5 workers** to be able to spawn some multi nodes clusters. This is also the reason why `kind` has been preferred as `minikube` another solution to run Kubernetes locally but handling only a managin a single worker.

📘 **6a. Execute the following command to create a kubernetes cluster**
```bash
kind create cluster --name kind-cassandra --config ./0-setup-your-cluster/01-kind-config.yaml
```

📗 **Expected output**
```
Creating cluster "kind-cassandra" ...
 ✓ Ensuring node image (kindest/node:v1.17.0) 🖼
 ✓ Preparing nodes 📦 📦 📦 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
 ✓ Joining worker nodes 🚜 
Set kubectl context to "kind-kind-cassandra"
You can now use your cluster with:
kubectl cluster-info --context kind-kind-cassandra
Have a question, bug, or feature request? Let us know! https://kind.sigs.k8s.io/#community 🙂
```

📘 **6b. Execute the following command to visualize the list of your clusters**
```bash
# Show the new created cluster
kind get clusters
```
📗 **Expected output**
```
kind-cassandra
```

📘 **6c. Execute the following command to link `kubectl` with our new cluster**
```bash
kubectl cluster-info --context kind-kind-cassandra
```
📗 **Expected output**
```
Kubernetes master is running at https://127.0.0.1:45451
KubeDNS is running at https://127.0.0.1:45451/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

📘 **6d. Execute the following command to list nodes in k8s cluster**
```bash
kubectl get nodes
```
📗 **Expected output**
```
NAME                           STATUS   ROLES    AGE    VERSION
kind-cassandra-control-plane   Ready    master   2m4s   v1.17.0
kind-cassandra-worker          Ready    <none>   86s    v1.17.0
kind-cassandra-worker2         Ready    <none>   88s    v1.17.0
kind-cassandra-worker3         Ready    <none>   88s    v1.17.0
kind-cassandra-worker4         Ready    <none>   88s    v1.17.0
kind-cassandra-worker5         Ready    <none>   88s    v1.17.0
```

## 7. Create namespace and storageClass

Cass-operator is built to watch over pods running Casandra or DSE in a Kubernetes namespace. We need to create a namespace for the cluster. For the rest of this guide, we will be using the namespace `cass-operator`. You can pick any name you like but you would have to change the commands accordingly.

📘 **7a. Execute the following command to create the namespace**
```bash
kubectl create ns cass-operator
```
📗 **Expected output**
```
namespace/cass-operator created
```

Kubernetes uses the `StorageClass` resource as an abstraction layer between pods needing persistent storage and the storage resources that a specific Kubernetes cluster can provide. We recommend using the fastest type of networked storage available. Let's create one for your environment.

📘 **7b. Execute the following command to list existing storageClass**
```bash
kubectl get storageclass
```
📗 **Expected output**
```bash
NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  11m
```

📘 **7c. Execute the following command describe `default` storageClass**
```bash
kubectl describe storageclass standard
```
📗 **Expected output** (may vary)
```
Name:            standard
IsDefaultClass:  Yes
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"},"name":"standard"},"provisioner":"rancher.io/local-path","reclaimPolicy":"Delete","volumeBindingMode":"WaitForFirstConsumer"}
,storageclass.kubernetes.io/is-default-class=true
Provisioner:           rancher.io/local-path
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     WaitForFirstConsumer
Events:                <none>
```

We are are interested in a few fields to intialize the `yaml` to create a storageClass. You can find more information in the [Reference Documentation](https://kubernetes.io/docs/concepts/storage/storage-classes/) and the [dedicated page](https://kubernetes.io/docs/tasks/administer-cluster/change-default-storage-class/#changing-the-default-storageclass) to edit those.

One thing thing to note here is `volumeBindingMode: WaitForFirstConsumer`. The default value is `Immediate` and should not be used. It can prevent Cassandra pods from being scheduled on a worker node. If a pod fails to run and its status reports a message like, had volume node affinity conflict, then check the `volumeBindingMode` of the `StorageClass` being used

This file should be adapted tor reflect your *Kubernetes* environment. We have several samples in the repository for [kind](02-storageclass-kind.yaml), [Minikube](03-storageclass-minikube.yaml) and [GKE](04-storageclass-gke.yaml).
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
  name: server-storage
provisioner: rancher.io/local-path
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

📘 **7d. Execute the following command to create a new `storageClass` named `server-storage`**
For the rest of this guide, we would assume you have defined a `StorageClass` and named it `server-storage`. 
```bash
kubectl -n cass-operator apply -f ./0-setup-your-cluster/02-storageclass-kind.yaml
```

📘 **7e. Execute the following command to list the storageClass**
```
kubectl -n cass-operator get storageClass
```
📗 **Expected output**
```
NAME                       PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
server-storage (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  25m
standard (default)         rancher.io/local-path   Delete          WaitForFirstConsumer   false                  88m
```

## Congratulations, your are done for this exercise.

![OK](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/welldone.jpg?raw=true)

