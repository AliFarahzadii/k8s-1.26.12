# Offline installation for Kubernetes 1.26 (Ubuntu server 20.4)*

**On machine with internet access:**

Update the apt package index and install packages needed to use the Kubernetes apt repository:

```
sudo apt-get update
```

\# apt-transport-https may be a dummy package; if so, you can skip that package

```
sudo apt-get install -y apt-transport-https ca-certificates curl
```

Download the public signing key for the Kubernetes package repositories. The same signing key is used for all repositories so you can disregard the version in the URL:

```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.26/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Add the appropriate Kubernetes apt repository. Please note that this repository have packages only for Kubernetes 1.26; for other Kubernetes minor versions, you need to change the Kubernetes minor version in the URL to match your desired minor version (you should also check that you are reading the documentation for the version of Kubernetes that you plan to install).

\# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list

```
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.26/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Update the apt package index, install kubelet, kubeadm and kubectl, and pin their version:

```
sudo apt-get update

sudo apt install --download-only -o=dir::cache=/home/kuber kubeadm
```

it will  be download all packages and dependencies . so copy all the downloaded packages to offline machine and install the packages .

then we need to pull and export some images so by using below command will show  all images we need :

```
kubeadm config images list

```

the output is :
```
registry.k8s.io/kube-apiserver:v1.26.12

registry.k8s.io/kube-controller-manager:v1.26.12

registry.k8s.io/kube-scheduler:v1.26.12

registry.k8s.io/kube-proxy:v1.26.12

registry.k8s.io/pause:3.9

registry.k8s.io/etcd:3.5.9-0

registry.k8s.io/coredns/coredns:v1.9.3
```

so we use **Cloud Shell**  (<https://shell.cloud.google.com/> ) to pull all images .

```
docker pull registry.k8s.io/coredns/coredns:v1.9.3
```

after pulling all images we export all images individually in tar files :

```
docker save -o filename.tar <repo>:<tag>
```

then download all tar files and copy to offline machine.

**On offline machine :**

To install kubeadm , kubelet , kubectl we download all packages in online machine and copied in a directory in offline machine use this command :

```
sudo dpkg -i \*.deb
```

to import the images that already copied in a directory use below command : 
```
ctr --namespace k8s.io image import  imageName.tar
```

then initiate the kubernetes cluster

- on worker node just import kube-proxy and pause image.
- the pause version must be 3.9 check the pause version in this path : /etc/containerd/config.toml.

![Aspose Words 5ffa7e6f-6edd-4305-9a46-e168142f5cfb 002](https://github.com/AliFarahzadii/k8s-1.26.12/assets/152854585/c037ca72-012a-4ca5-aa45-d2d81ebffb05)
