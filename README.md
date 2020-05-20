This has been done on the following ISO. 

- CentOS-7-x86_64-Minimal-2003.iso

2 vms. 

1 master and one node.

Both with the specs:

- 6GB, 2 CPU, 20GB HDD

-----------
Docker Configuration
Docker version: 19.03.9

Cgroup Driver: systemd ( not cgroups )

- add "--exec-opt native.cgroupdriver=systemd" in the ExecStart in docker.service systemd unit file. 
------------

------------
Kubernetes Configuration
Kubeadm, Kubectl and Kubelet version: 1.18 ( git version v1.18.3 )
------------

------------
SDN Configuration

I am using Calico.
-----------
Node Configuration

1. As a requirement for your Linux Node’s iptables to correctly see bridged traffic, 

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system

2. Kubeadm does not need swap. Disable all swap and comment entry in fstab as well.

3. Make sure that the br_netfilter module is loaded. 

sudo modprobe br_netfilter

4. Disable SELinux
-----------

--------------
Firewall Ports

On master(all tcp)
#########
6443
2379-2380
10250	 
10251	
10252	
9099(calico)
#########

On worker(all tcp)
#########
10250
30000-32767
9099(calico)
#########

Worst case scenario, if you still think it is not working after opening all the ports,

firewall-cmd --add-port=0-65535/tcp --permanent
firewall-cmd --add-port=0-65535/udp --permanent
firewall-cmd --reload

------------
Kubeadm Init
------------
kubeadm init --pod-network-cidr=10.244.0.0/16

This will take some time, let it work.
Follow the logs to setup the directory and kubeconfig files. 

------------
POST INSTALL ( VERY IMPORTANT )

After you have done the kubeadm init, the coredns will not be running as there is no cni in the cluster yet. They are dependent on the CNI. We are going to use calico.

kubectl apply -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml

After this, test the coredns pods again, they should be ready by now.
------------

------------
Adding a new node
------------

The installation of kubeadm init command will give you a token you can use on the node to join more than one worker nodes in your cluster.

Before you run the command on a node, make sure the following.

1. Have connectivity. Get a local dns server running ( recommended ).
2. Have the same version of docker and kube* binaries installed. (look above)
3. disable swap (look above)
4. configure docker to use systemd cgroup driver (look above)
5. add respective port numbers. (look above)
