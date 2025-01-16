
Launch 3 EC2 instances pass the below script in user data.
```sh
#/bin/bash 
tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
modprobe overlay
modprobe br_netfilter
tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
 sysctl --system
mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

apt-get update && apt-get install -y containerd.io

mkdir -p /etc/containerd
containerd config default>/etc/containerd/config.toml
sed -e 's/SystemdCgroup = false/SystemdCgroup = true/g' -i /etc/containerd/config.toml

systemctl daemon-reload
systemctl restart containerd
systemctl enable containerd 
systemctl status containerd

apt-get update && apt-get install -y apt-transport-https ca-certificates curl


curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.27/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.27/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

apt-get update && apt-get install -y kubelet kubeadm kubectl

apt-mark hold kubelet kubeadm kubectl
systemctl enable kubelet

```


```sh
#!/bin/bash 

apt-get update
apt-get install -y git wget

# Install Docker
apt-get install -yq \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
    

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
apt-key fingerprint 0EBFCD88
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
   
apt-get update

apt-get install -yq docker-ce docker-ce-cli containerd.io

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

sudo cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update -y

sudo apt-get install -y  kubelet kubeadm kubectl kubernetes-cni nfs-common

# Install Docker Compose 
sudo curl -L "https://github.com/docker/compose/releases/download/1.28.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose


sudo sysctl net.bridge.bridge-nf-call-iptables=1
sudo swapoff -a

sudo rm -rf /var/lib/kubelet/*

sudo apt-get install nfs-common -y
```
