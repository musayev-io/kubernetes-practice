Kubernetes Notes
================

Installation Instructions
-------------------------

### CentOS Installation

*Prerequisites* : Kubernetes will not install if you have swap running, so we need to turn that off: - `sudo swapoff -a` - In `/etc/fstab`, comment out line `/root/swap swap swap sw 0 0`

1.	**Install Docker**

	1.1. `yum update -y && yum install -y docker` 1.2 `systemctl enable docker`

2.	**Install Kubernetes**

	2.1 Add Kubernetes Repository Key

	```
	 cat <<EOF > /etc/yum.repos.d/kubernetes.repo
	 [kubernetes]
	 name=Kubernetes
	 baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
	 enabled=1
	 gpgcheck=1
	 repo_gpgcheck=1
	 gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
	 EOF
	```

	2.2 Turn off SELINUX

	-	`setenforce 0`
	-	In `/etc/selinux/config`, change from `SELINUX=enforcing` to `SELINUX=permissive`

	2.3 Install Kubernetes packages

	-	`yum install -y kubelet kubeadm kubectl`

	2.4 Start Kubernetes service and persist on reboot

	-	`systemctl enable kubelet && systemctl start kubelet`

	2.5 Apply Network Settings for Pod Intercommunication

	```
	  cat <<EOF >  /etc/sysctl.d/k8s.conf
	  net.bridge.bridge-nf-call-ip6tables = 1
	  net.bridge.bridge-nf-call-iptables = 1
	  EOF
	```

	2.6 Apply Changes

	-	`sysctl --system`

3.	**Configure Kubernetes**

	3.1 `kubeadm init--pod-network-cidr=10.255.0.0/16`

	3.2 *Make a note of the code that you'll need to run on your Worker nodes to join them to the cluster. It will look something like this*

	-	`kubeadm join 172.31.25.30:6443 --token z7lmk4.v3fhppz3xg63c7uy --discovery-token-ca-cert-hash sha256:e458b4b0519674db89f4678315f11dc29366b0c7d5c7eb43d9125d5838f55e19`

	3.3 Create and Configure Kubernetes Home Directory

	```
	  mkdir -p $HOME/.kube
	  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
	  sudo chown $(id -u):(id -g) $HOME/.kube/config
	```

4.	**Install Flannel for Pod Networking**

	4.1 `kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml`

5.	**Confirm Your Master Node is Online**

	5.1 `kubectl get nodes`

	-	You should see the Hostname of your server in Master Role and Ready status.

### Firewall Rules

#### Master Nodes

| Protocol | Port      | Program                 |
|----------|-----------|-------------------------|
| TCP      | 6443\*    | Kubernetes API Server   |
| TCP      | 2379-2380 | etc Server Client API   |
| TCP      | 10250     | Kubelet API             |
| TCP      | 10251     | kube-scheduler          |
| TCP      | 10252     | kube-controller-manager |
| TCP      | 10255     | Read-Only Kubelet API   |

#### Worker Nodes

| Protocol | Port        | Program               |
|----------|-------------|-----------------------|
| TCP      | 10250       | Kubelet API           |
| TCP      | 10255       | Read-Only Kubelet api |
| TCP      | 30000-32767 | NodePort Services     |
