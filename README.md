####################################################### kubeadm installation on ubuntu machine ###################################

########Disable swap on all the Nodes############

sudo swapoff -a

###############Set up required sysctl params, these persist across reboots####################

cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

#######Execute the following commands to enable overlayFS & VxLan pod communication#########

sudo modprobe overlay
sudo modprobe br_netfilter


############Set up required sysctl params, these persist across reboots.################

cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

#############Reload the parameters#####################

sudo sysctl --system

####################Enable cri-o repositories for version 1.23######################

OS="xUbuntu_20.04"

VERSION="1.23"

cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /
EOF
cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list
deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /
EOF

########################Add the gpg keys.###############################

curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -

###################Update and install crio and crio-tools.###############################

sudo apt-get update
sudo apt-get install cri-o cri-o-runc cri-tools -y

##################reload the systemd configurations and enable cri-o.##############################

sudo systemctl daemon-reload
sudo systemctl enable crio --now

#################Install the required dependencies.#####################
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

#################Add the GPG key and apt repository###################################

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

#################Update apt and install the latest version of kubelet, kubeadm, and kubectl#########

sudo apt-get update -y
sudo apt-get install -y kubelet kubeadm kubectl
