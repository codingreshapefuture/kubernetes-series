Trên tất cả các node sẽ cài các thành phần: docker, kubelet, kubeadm và kubectl
+ Docker/Containerd: để làm môi trường chạy các container runtime.
+ kubeadm: Được sử dụng để thiết lập cụm cluster cho K8S. Các tài liệu chuyên môn gọi kubeadm là bột bootstrap (tools đóng gói để tự động làm việc gì đó)
+ kubelet: Là thành phần chạy trên các host, có nhiệm vụ kích hoạt các pod và container trong cụm Cluser của K8S.
+ kubectl: Là công cụ cung cấp CLI để tương tác với K8S.

Open ports phụ thuộc:
```
TCP/2379: etcd endpoint
TCP/2380: etcd member
TCP/10250: kubelet API
TCP/6443: apiserver
```

----------

Đổi hostname (optional):
```
hostnamectl set-hostname k8s-manager-m1
exec bash
```

Tắt swap và Forward IPv4 (all nodes):
```
sudo swapoff -a
sudo sed -i '/swap/s/^\(.*\)$/#\1/g' /etc/fstab
sudo mount -a
free -h
sudo modprobe overlay
sudo modprobe br_netfilter
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
```

Cài đặt container runtime (all nodes):

Có thể cài docker.io hoặc containerd.io cho nhẹ (cài theo docs là được)
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y containerd.io
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

Cài đặt các gói Kubernetes (all nodes):
```
sudo apt-get update
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
```

===Tạo K8s cluster (1 master node) (k8s-master):

Đứng trên node k8s-master thực hiện lệnh dưới để thiết lập cluster
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address <ip_master_node>
```

===Với mô hình đa master node:
Khởi tạo Cluster:
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --control-plane-endpoint <ip_LB:6443> --upload-certs
```

Nếu lỗi có thể reset bằng:
```
kubeadm reset -f
```

Xem lại token để join node trên k8s master:
```
kubeadm token create --print-join-command
```

Cert mời vào control-plane có hạn 2h, có thể tạo lại bằng cách:
```
sudo kubeadm init phase upload-certs --upload-certs
```

Tạo lại SHA Key:
```
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```

Các master-node có thể chạy lệnh sau để join cluster
```
kubeadm join <ip-master>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<SHA-KEY> --control-plane --certificate-key <CERTIFICATE-KEY>
```

Các worker-node chạy lệnh sau để join cluster:
```
kubeadm join <ip-master>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<SHA-KEY>
```

===

Thiết lập cơ bản (master nodes):

Làm theo hướng dẫn để sử dụng cluster:

Với root user:
```
export KUBECONFIG=/etc/kubernetes/admin.conf
```

Với normal user:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Kiểm tra:
```
kubectl cluster-info
kubectl get nodes
```

Cài đặt Pod network plugin (k8s-master):

Mặc định sau khi tạo ở bước này các Node status vẫn là NotReady, cần thêm network plugin để chúng giao tiếp với nhau, có nhiều loại nhưng ở đây sử dụng flannel
```
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

Cho phép Pods có thể chạy trên master node:
```
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
kubectl get nodes
```

Đánh label cho các node thành master (optional):
```
kubectl label node <node-name> node-role.kubernetes.io/master=true
```

Quản lý cluster đã triển khai:
```
kubectl get nodes -owide
kubectl config view
kubectl cluster-info
kubectl get pod -A
```
