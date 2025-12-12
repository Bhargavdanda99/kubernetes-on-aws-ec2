# kubernetes-on-aws-ec2
Installing k8s on AWS Ec2 servers
**LOCAL K8 Setup Using  Kubeadm**
Create 3 VMs with Port as below.

<img width="751" height="346" alt="image" src="https://github.com/user-attachments/assets/d535bc9b-8ed0-42d5-b429-83c14d4d58fd" />
<img width="448" height="61" alt="image" src="https://github.com/user-attachments/assets/437c9aa7-c4e4-4830-8548-fbb2b06fc57e" />
Step 1: Run on All Nodes (Master + Workers)
# 1. Update 
sudo apt update && sudo apt upgrade -y

# 2. Disable Swap (required by kubelet)
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
 
# 3. Load required kernel modules
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# 4. Set sysctl parameters
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system

Step 2: Install containerd
# 1. Install containerd dependencies
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

# 2. Add Docker GPG key and repo
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 3. Install containerd
sudo apt update && sudo apt install -y containerd.io

# 4. Generate default config
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null

# 5. Enable systemd cgroup driver
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

# 6. Restart containerd
sudo systemctl restart containerd
sudo systemctl enable containerd

# 1. Add Kubernetes GPG key and repo
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | \
sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | \
sudo tee /etc/apt/sources.list.d/kubernetes.list

# 2. Install kubeadm, kubelet, kubectl
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

Step 4: Initialize the Control Plane (Master)
sudo kubeadm init --pod-network-cidr=192.168.0.0/16

Step 5: Configure kubectl on Master
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

**Step 5: Configure kubectl on Master**
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

**Step 6: Join Worker Nodes**
On each worker node (from the join command you copied earlier):
sudo kubeadm join <MASTER-IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>



