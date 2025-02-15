sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab

cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system

sudo apt-get update -y
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg

sudo apt-get install -y containerd

sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
sudo systemctl restart containerd
sudo systemctl enable containerd

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo tee /etc/apt/keyrings/kubernetes-apt-keyring.asc > /dev/null
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.asc] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update -y
sudo apt-get install -y kubelet kubeadm kubectl

sudo systemctl enable kubelet
sudo systemctl start kubelet

# Running only master node

sudo sysctl -w net.ipv4.ip_forward=1

sudo kubeadm init --pod-network-cidr=192.168.1.0/16

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubeadm token create --print-join-command

kubeadm join 172.16.129.158:6443 --token qwncff.djsmq401jvm1a69o --discovery-token-ca-cert-hash sha256:509703ff26c11caca254b189fc3d47d25996c940b12a36f56bef0c18b494e9aa

kubectl get nodes

Check API Server Logs

kubectl logs -n kube-system kube-apiserver-$(hostname)

Check node

journalctl -u kubelet -f

journalctl -u containerd/docker -f

kubectl logs <pod-name> -n <namespace>

kubectl logs -f -l app=<app-label> -n <namespace>

Debug pod
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> --previous -n <namespace> # Logs from last restart
kubectl exec -it <pod-name> -- /bin/sh # Access container shell
