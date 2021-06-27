Vagrant.configure("2") do |config|
  config.vm.provider :virtualbox do |v|
    v.memory = 2048
    v.cpus = 2
  end

  config.vm.provision :shell, privileged: true, inline: $install_common_tools

  config.vm.define :master do |master|
    master.vm.box = "ubuntu/xenial64"
    master.vm.hostname = "master"
    master.vm.network :private_network, ip: "10.0.0.10"
    master.vm.provision :shell, privileged: false, inline: $provision_master_node
  end

  %w{worker1 worker2 worker3}.each_with_index do |name, i|
    config.vm.define name do |worker|
      worker.vm.box = "ubuntu/xenial64"
      worker.vm.hostname = name
      worker.vm.network :private_network, ip: "10.0.0.#{i + 11}"
      worker.vm.provision :shell, privileged: false, inline: <<-SHELL
sudo /vagrant/join.sh
echo 'Environment="KUBELET_EXTRA_ARGS=--node-ip=10.0.0.#{i + 11}"' | sudo tee -a /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
sudo systemctl daemon-reload
sudo systemctl restart kubelet
SHELL
    end
  end

  config.vm.provision "shell", inline: $install_multicast
end


$install_common_tools = <<-SCRIPT
# bridged traffic to iptables is enabled for kube-router.
cat >> /etc/ufw/sysctl.conf <<EOF
net/bridge/bridge-nf-call-ip6tables = 1
net/bridge/bridge-nf-call-iptables = 1
net/bridge/bridge-nf-call-arptables = 1
EOF

# disable swap
swapoff -a
sed -i '/swap/d' /etc/fstab

# Install kubeadm, kubectl and kubelet
export DEBIAN_FRONTEND=noninteractive
apt-get -qq install ebtables ethtool
apt-get -qq update
apt-get -qq install -y docker.io apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get -qq update
apt-get -qq install -y kubelet kubeadm kubectl
SCRIPT

$provision_master_node = <<-SHELL
OUTPUT_FILE=/vagrant/join.sh
rm -rf $OUTPUT_FILE

# Start cluster
sudo kubeadm init --apiserver-advertise-address=10.0.0.10 --pod-network-cidr=10.244.0.0/16
sudo kubeadm token create --print-join-command > ${OUTPUT_FILE}
chmod +x $OUTPUT_FILE

# Configure kubectl
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
sudo cp $HOME/.kube/config /vagrant

# Fix kubelet IP
echo 'Environment="KUBELET_EXTRA_ARGS=--node-ip=10.0.0.10"' | sudo tee -a /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

# Configure cni plugin
kubectl apply -f https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')

# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.0.4/manifests/install.yaml
kubectl apply -n argocd -f https://raw.githubusercontent.com/kszpakowski/argo-apps/master/app-of-apps.yml

sudo systemctl daemon-reload
sudo systemctl restart kubelet
SHELL

$install_multicast = <<-SHELL
apt-get -qq install -y avahi-daemon libnss-mdns
SHELL