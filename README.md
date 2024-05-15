# --------- kubeadm install rhel9 ----------------

systemctl disable firewalld
systemctl stop firewalld
----------------------------------------------------------------------------------------

swapoff -a; sed -i '/swap/d' /etc/fstab
----------------------------------------------------------------------------------------

sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
----------------------------------------------------------------------------------------

sudo modprobe br_netfilter
sudo modprobe ip_vs
sudo modprobe ip_vs_rr
sudo modprobe ip_vs_wrr
sudo modprobe ip_vs_sh
sudo modprobe overlay

----------------------------------------------------------------------------------------
cat > /etc/modules-load.d/kubernetes.conf << EOF
br_netfilter
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
overlay
EOF

----------------------------------------------------------------------------------------
cat > /etc/sysctl.d/kubernetes.conf << EOF
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

----------------------------------------------------------------------------------------

sysctl --system

----------------------------------------------------------------------------------------

sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
----------------------------------------------------------------------------------------

sudo dnf makecache

----------------------------------------------------------------------------------------
sudo dnf -y install containerd.io

----------------------------------------------------------------------------------------

sudo sh -c "containerd config default > /etc/containerd/config.toml" ; cat /etc/containerd/config.toml

----------------------------------------------------------------------------------------

sudo vim /etc/containerd/config.toml
	- SystemdCgroup = true

----------------------------------------------------------------------------------------
sudo systemctl enable --now containerd.service

----------------------------------------------------------------------------------------

sudo systemctl reboot
----------------------------------------------------------------------------------------

cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF

----------------------------------------------------------------------------------------

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

sudo systemctl enable --now kubelet

----------------------------------------------------------------------------------------


vim /etc/hosts      <--- add all systems
192.168.169.134 	master.example.com   
192.168.169.136 	worker1.example.com    
192.168.169.137	        worker2.example.com

========================================================================================

Only for masternode
----------------------

kubeadm config images pull
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

only user
=========
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

root user
===========
export KUBECONFIG=/etc/kubernetes/admin.conf

kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml

curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml

wget https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml

sed -i 's/cidr: 192\.168\.0\.0\/16/cidr: 10.244.0.0\/16/g' custom-resources.yaml

kubectl create -f custom-resources.yaml

for join worker node
----------------------
sudo kubeadm token create --print-join-command
----------------------------------------------------------------------------------------
