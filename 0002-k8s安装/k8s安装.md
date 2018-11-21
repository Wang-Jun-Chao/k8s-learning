```
# 关闭防火墙
$ sudo systemctl stop firewalld

# 开机禁用 
$ sudo systemctl disable firewalld

禁用SELINUX
$ sudo setenforce 0 

$ sudo vim /etc/selinux/config
SELINUX=permissive 

$ sudo vim /etc/sysctl.conf
# 开启ipv4转发，允许内置路由
net.ipv4.ip_forward = 1  

# 写入后执行如下命令生效：
$ sudo sysctl -p          

# 第一步
$ vim /etc/yum.repos.d/kubernetes.repo

# 内容
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg


# 第二步
yum install -y docker kubelet kubeadm kubectl kubernetes-cni

# 第三步
systemctl enable docker && systemctl start docker

# 第四步
systemctl enable kubelet && systemctl start kubelet

# 第五步：下载Kubernetes 的相关镜像
docker pull warrior/pause-amd64:3.0
docker tag warrior/pause-amd64:3.0 gcr.io/google_containers/pause-amd64:3.0

docker pull warrior/etcd-amd64:3.0.17
docker tag warrior/etcd-amd64:3.0.17 gcr.io/google_containers/etcd-amd64:3.0.17

docker pull warrior/kube-apiserver-amd64:v1.6.0
docker tag warrior/kube-apiserver-amd64:v1.6.0 gcr.io/google_containers/kube-apiserver-amd64:v1.6.0

docker pull warrior/kube-scheduler-amd64:v1.6.0
docker tag warrior/kube-scheduler-amd64:v1.6.0 gcr.io/google_containers/kube-scheduler-amd64:v1.6.0

docker pull warrior/kube-controller-manager-amd64:v1.6.0
docker tag warrior/kube-controller-manager-amd64:v1.6.0 gcr.io/google_containers/kube-controller-manager-amd64:v1.6.0

docker pull warrior/kube-proxy-amd64:v1.6.0
docker tag warrior/kube-proxy-amd64:v1.6.0 gcr.io/google_containers/kube-proxy-amd64:v1.6.0

docker pull gysan/dnsmasq-metrics-amd64:1.0
docker tag gysan/dnsmasq-metrics-amd64:1.0 gcr.io/google_containers/dnsmasq-metrics-amd64:1.0

docker pull warrior/k8s-dns-kube-dns-amd64:1.14.1
docker tag warrior/k8s-dns-kube-dns-amd64:1.14.1 gcr.io/google_containers/k8s-dns-kube-dns-amd64:1.14.1

docker pull warrior/k8s-dns-dnsmasq-nanny-amd64:1.14.1
docker tag warrior/k8s-dns-dnsmasq-nanny-amd64:1.14.1 gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.1

docker pull warrior/k8s-dns-sidecar-amd64:1.14.1
docker tag warrior/k8s-dns-sidecar-amd64:1.14.1 gcr.io/google_containers/k8s-dns-sidecar-amd64:1.14.1

docker pull awa305/kube-discovery-amd64:1.0 
docker tag awa305/kube-discovery-amd64:1.0 gcr.io/google_containers/kube-discovery-amd64:1.0

docker pull gysan/exechealthz-amd64:1.2 
docker tag gysan/exechealthz-amd64:1.2 gcr.io/google_containers/exechealthz-amd64:1.2

# 第六步：关闭系统的Swap
swapoff -a
设置永久关闭swap
修改/etc/fstab中内容，将swap哪一行用#注释掉。
删除etcd 
yum erase etcd. 
删除etcd文件夹 mv /var/lib/etcd /var/lib/etcd.bak

# 第六步：运行kubeadm init 安装Master
kubeadm init --kubernetes-version=1.12.0


docker tag docker.io/coredns/coredns:1.2.2  k8s.gcr.io/coredns:1.2.2

```