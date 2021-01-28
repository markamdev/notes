# Kubernetes cluster launching on Raspberry Pi running Ubuntu Server 20.04

This document describes all steps needed to successfully launch K8s cluster using KubeAdm tool.

## Prerequisities

This document assumes that Docker is already installed and available for *ubuntu* user (user added to *docker* group).

All presented steps have been executed on following system configuration:

* Raspberry Pi 4 with 4GB of RAM
* 32GB of storage on micro SD card
* Ubuntu Server 20.04

### Kernel options

Some kernel options regarding memory and swap managements are needed. These options are

```config
cgroup_enable=cpuset cgroup_enable=memory swapaccount=1
```

and should be added at end of */boot/firmware/cmdline.txt* file.

System has to be rebooted after this update.

### Docker options

It is necessary to have *systemd* seet as cgroup manager for Docker. To enable this create */etc/docker/daemon.json* file (or update it's content if already exists) with following setting:

```json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
```

Restart docker after this operation using

```bash
systemctl restart docker
```

### Tools and libraries installation

Following set of tools/libraries have to be installed on your system to successfully launch cluster:

* conntrack
* kubelet
* kubeadm

**conntrack installation**

*conntrack* can be installed using *apt* command for standard Ubuntu repositories:

```bash
sudo apt install conntrack
```

**not recommended kubelet and kubeadm installation**

For *kubelet* and *kubeadm* you can use *snap* package manager with commands below:

```bash
sudo snap install kubelet --classic
sudo snap install kubeadm --classic
```

To launch cluster kubelet has to be up and running. As it is snap-based installation it should be started using command:

```bash
sudo snap start --enable kubelet
```

This can fail due to *apparmor* policies. Additional package with apparmor config and tools are needed:

```bash
sudo apt install -y apparmor-profiles apparmor-utils
```

Also to properly launch *kubelet* without "incorrect cgroup driver error" one should add `--cgroup-driver=systemd` in file `/etc/systemd/system/snap.kubelet.daemon.service`.

**recommended kubelet and kubeadm installation**

To install *kubeadm*, *kubelet* and *kubectl* as a standard *apt* packages use commands below:

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
```

Also mark *kube\** packages to not be upgraded:

```bash
sudo apt-mark hold kubelet kubeadm kubectl
```

## Cluster installation

On first node (control-plane, master node - call it as you wish) call *kubeadm init* with necessary options. One example you can find below:

```bash
kubeadm init --control-plane-endpoint machine1.domain  --node-name machine1 --pod-network-cidr 10.30.0.0/16
```

This will result with output similar to below:

```config
our Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join machine1.domain:6443 --token aaaabbbbcccc \
    --discovery-token-ca-cert-hash sha256:0123456789abcdf0123456789abcdf0123456789abcdf0123456789abcdf0123 \
    --control-plane

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join machine1.domain:6443 --token aaaabbbbcccc \
    --discovery-token-ca-cert-hash sha256:0123456789abcdf0123456789abcdf0123456789abcdf0123456789abcdf0123
```

Use provided *join* commands to add new nodes to your cluster.

Token can be obtained any time using `kubeadm token ...` command.

## Post-installation job

"Raw" cluster does not have any internal network configuration. To solve this issue one can apply for example *flannel* plugin:

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

More information about this can be found in documentation related to [*Container Network Interface*](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#cni).
