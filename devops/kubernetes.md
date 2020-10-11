# Kubernetes

## Setup using kubeadm


> Reference: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

Requirements:
* One or more machines running a deb/rpm-compatible Linux OS; for example: Ubuntu or CentOS.
* 2 GiB or more of RAM per machine--any less leaves little room for your apps.
* At least 2 CPUs on the machine that you use as a control-plane node.
* Full network connectivity among all machines in the cluster. You can use either a public or a private network.

You also need to use a version of kubeadm that can deploy the version of Kubernetes that you want to use in your new cluster.



## Infrastructure

For this lab, I'm going to use 3 hosts. 1 master and 2 workers.

![kube-cluster](https://rancher.com/learning-paths/03-introduction-to-kubernetes-architecture/media/single_node_cluster.png)

### Installing

> The following commands, need to be running on all hosts.

```
$ sudo setenforce 0

$ sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

```
$ cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

$ sudo yum -y update

$ sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```

Configuring Docker
```
$ sudo yum install -y epel-release yum-utils vim device-mapper-persistent-data lvm2

$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

$ sudo yum install -y containerd.io-1.2.13 docker-ce-19.03.11 docker-ce-cli-19.03.11

$ sudo mkdir /etc/docker

$ sudo mkdir -p /etc/systemd/system/docker.service.d

$ cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF
$ sudo usermod -aG docker vagrant

$ sudo systemctl daemon-reload

$ sudo systemctl restart docker

$ sudo systemctl enable docker
```

### Starting the cluster

```
sudo kubeadm config images pull

sudo kubeadm init --pod-network-cidr=192.168.0.0/16

mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config
```




