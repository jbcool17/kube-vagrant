# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
# MASTER
  (1..1).each do |i|
      config.vm.define "kube-master-#{i}" do |node|
        node.vm.box = "ubuntu/xenial64"
        node.vm.hostname = "kube-master-#{i}"
        # node.vm.network "private_network", ip: "192.168.33.1#{i}", auto_config: false
        node.vm.network "public_network", bridge: "en0: Wi-Fi (AirPort)"
        node.vm.provision "shell", inline: <<-SHELL
echo "Swap..."
# kubelet requires swap off
swapoff -a
# keep swap off after reboot
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# SETTING UP KUBERNETES
echo "installing docker"
apt-get update
apt-get install -y \
  apt-transport-https \
  ca-certificates \
  curl \
  software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository \
 "deb https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
 $(lsb_release -cs) \
 stable"
apt-get update && apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 17.03 | head -1 | awk '{print $3}')

echo "installing kubernetes"
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF

iptables -I INPUT -j ACCEPT

apt-get update
apt-get install -y kubelet kubeadm kubectl

# DigitalOcean without firewall (IP-in-IP allowed) - or any other cloud / on-prem that supports IP-in-IP traffic
# echo "deploying kubernetes (with calico)..."
# kubeadm init --pod-network-cidr=192.168.0.0/16 # add --apiserver-advertise-address="ip" if you want to use a different IP address than the main server IP
# export KUBECONFIG=/etc/kubernetes/admin.conf
# kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
# kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml


# DigitalOcean with firewall (VxLAN with Flannel) - could be resolved in the future by allowing IP-in-IP in the firewall settings
echo "deploying kubernetes (with canal)..."
kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address="$(ip addr | grep  "enp0s8" | grep inet | awk '{ print $2 }' | cut -d '/' -f 1)" # add --apiserver-advertise-address="ip" if you want to use a different IP address than the main server IP
export KUBECONFIG=/etc/kubernetes/admin.conf
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/canal/rbac.yaml
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/canal/canal.yaml

# create .kube/config
sudo -H -u vagrant bash -c 'mkdir -p ~vagrant/.kube'
sudo -H -u vagrant bash -c 'sudo cp -i /etc/kubernetes/admin.conf ~vagrant/.kube/config'
sudo -H -u vagrant bash -c 'sudo chown vagrant: ~vagrant/.kube/config'
SHELL
      end
    end

# NODES
  (1..2).each do |i|
      config.vm.define "kube-node-#{i}" do |node|
        node.vm.box = "ubuntu/xenial64"
        node.vm.hostname = "kube-node-#{i}"
        # node.vm.network "private_network", ip: "192.168.33.2#{i}", auto_config: false
        node.vm.network "public_network", bridge: "en0: Wi-Fi (AirPort)"
        node.vm.provision "shell", inline: <<-SHELL
# kubelet requires swap off
swapoff -a
# keep swap off after reboot
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
echo "installing docker"
apt-get update
apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository \
   "deb https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
   $(lsb_release -cs) \
   stable"
apt-get update && apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 17.03 | head -1 | awk '{print $3}')

iptables -I INPUT -j ACCEPT

echo "installing kubeadm and kubectl"
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
SHELL
      end
    end
end
