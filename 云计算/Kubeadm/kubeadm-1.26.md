[toc]


```
cat <<EOF >> /etc/hosts
192.168.3.201 k8s-master
192.168.3.202 k8s-worker01
EOF


cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# 设置所需的 sysctl 参数，参数在重新启动后保持不变
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# 应用 sysctl 参数而不重新启动
sudo sysctl --system

lsmod | grep br_netfilter
lsmod | grep overlay

sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```





## 安装容器运行时

### 环境准备

```bash
# 转发 IPv4 并让 iptables 看到桥接流量
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# 设置所需的 sysctl 参数，参数在重新启动后保持不变
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# 应用 sysctl 参数而不重新启动
sudo sysctl --system

# 检查 overlay 和 br_netfilter, 是否被加载
lsmod | grep br_netfilter
lsmod | grep overlay

# 确认 net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward 被设置为 1
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```

![image-20230409201353685](media/image-20230409201353685.png)

### Containerd

Containerd 或者 CRI-O 任意装一个就行

> 文档: https://github.com/containerd/containerd/blob/main/docs/getting-started.md

```bash
# 安装 contianerd
tar Cxzvf /usr/local containerd-1.6.20-linux-amd64.tar.gz
mkdir -p /usr/local/lib/systemd/system

cd /usr/local/lib/systemd/system
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service

systemctl daemon-reload
systemctl enable containerd

# 列出容器
/usr/local/bin/ctr -n k8s.io c ls
# 列出镜像
/usr/local/bin/ctr -n k8s.io i ls
```

```bash
# 安装 runc, conitanerd 安装包中不包含runc, 需要手动安装
# 下载 runc.amd64文件, https://github.com/opencontainers/runc/releases
install -m 755 runc.amd64 /usr/local/sbin/runc

# 安装 CNI plugins
# 下载 cni-plugins-<OS>-<ARCH>-<VERSION>.tgz 安装包, https://github.com/containernetworking/plugins/releases
mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.2.0.tgz

# 生成 containerd 配置文件
mkdir /etc/containerd
containerd config default > /etc/containerd/config.toml

vim /etc/containerd/config.toml
# 配置runc systemd cgroup驱动
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
# 设置阿里云地址
[plugins."io.containerd.grpc.v1.cri"]
  sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.6"
# 设置阿里云镜像加速器
[plugins."io.containerd.grpc.v1.cri".registry]
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
      endpoint = ["https://8aj710su.mirror.aliyuncs.com"]

# 启动 containerd
sudo systemctl restart containerd
```

### CRI-O

> 文档: https://github.com/cri-o/cri-o/blob/main/install.md#readme

## Kubeadm 部署 kubernetes

```bash
# 配置 aliyun k8s yum 源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# master 安装 kubeadm kubelet kubectl
yum install -y kubeadm-1.26.2 kubelet-1.26.2 kubectl-1.26.2
systemctl enable --now kubelet

# worker 安装 kubeadm kubelet
yum install -y kubeadm-1.26.2 kubelet-1.26.2
systemctl enable --now kubelet

# init master 节点
kubeadm init \
--image-repository registry.aliyuncs.com/google_containers \
--apiserver-advertise-address=192.168.3.201 \
--kubernetes-version=v1.26.2 \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16 \
--v=5

# init worker节点, PASS

# 安装flannel
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

# 检查flannel镜像状态
ctr -n k8s.io c ls
```

## 二进制安装 Kubernetes

### 手动生成证书

```shell
mkdir ssl && cd ssl

# 1. 生成一个 2048 位的 ca.key 文件
openssl genrsa -out ca.key 2048
# 2. 在 ca.key 文件的基础上，生成 ca.crt 文件（用参数 -days 设置证书有效期）
openssl req -x509 -new -nodes -key ca.key -subj "/CN=192.168.3.201" -days 10000 -out ca.crt

# 3. 生成一个 2048 位的 server.key 文件
openssl genrsa -out server.key 2048

# 4. 创建一个用于生成证书签名请求（CSR）的配置文件。
cat <<EOF>>csr.conf
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn

[ dn ]
C = CN
ST = BJ
L = BJ
O = k8s
OU = system
CN = 192.168.3.201

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster
DNS.5 = kubernetes.default.svc.cluster.local
IP.1 = 192.168.3.201
IP.2 = 192.168.3.201

[ v3_ext ]
authorityKeyIdentifier=keyid,issuer:always
basicConstraints=CA:FALSE
keyUsage=keyEncipherment,dataEncipherment
extendedKeyUsage=serverAuth,clientAuth
subjectAltName=@alt_names
EOF

# 5. 基于上面的配置文件生成证书签名请求
openssl req -new -key server.key -out server.csr -config csr.conf

# 6. 基于 ca.key、ca.crt 和 server.csr 等三个文件生成服务端证书
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key \
    -CAcreateserial -out server.crt -days 10000 \
    -extensions v3_ext -extfile csr.conf -sha256

# 7. 查看证书签名请求
openssl req  -noout -text -in ./server.csr

# 8. 查看证书
openssl x509  -noout -text -in ./server.crt
```

```shell
curl -L https://github.com/cloudflare/cfssl/releases/download/v1.5.0/cfssl_1.5.0_linux_amd64 -o cfssl
chmod +x cfssl
curl -L https://github.com/cloudflare/cfssl/releases/download/v1.5.0/cfssljson_1.5.0_linux_amd64 -o cfssljson
chmod +x cfssljson
curl -L https://github.com/cloudflare/cfssl/releases/download/v1.5.0/cfssl-certinfo_1.5.0_linux_amd64 -o cfssl-certinfo
chmod +x cfssl-certinfo

mkdir ~/bin
mv -v cfssl* ~/bin/
```

```shell
#!/bin/bash
# certificate.sh
mkdir /root/ssl && cd /root/ssl
cat > ca-config.json <<EOF
{
    "signing": {
        "default": {
            "expiry": "87600h"
        },
        "profiles": {
            "kubernetes": {
                "expiry": "87600h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        },
        "kcfg": {
            "usages": [
                "signing",
                "key encipherment",
                "client auth"
            ],
            "expiry": "87600h"
        }
    }
}
EOF


# --------------------------------------------------
# 生成ca.pem ca-key.pem
# --------------------------------------------------
cat > ca-csr.json <<EOF
{
    "CN": "kubernetes",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Beijing",
            "ST": "Beijing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca -


# --------------------------------------------------
# 生成 server.pem server-key.pem
# 配置说明:
# 集群节点IP: "192.168.30.130", "192.168.30.131", "192.168.30.132"
# kubernetes服务ip: "10.1.0.1", 一般是 kue-apiserver 指定的 service-cluster-ip-range 网段的第一个IP
# 集群域名: "kubernetes*"
# --------------------------------------------------
cat > server-csr.json <<EOF
{
    "CN": "kubernetes",
    "hosts": [
        "127.0.0.1",
		"192.168.3.201",
        "10.0.0.1",
        "kubernetes",
        "kubernetes.default",
        "kubernetes.default.svc",
        "kubernetes.default.svc.cluster",
        "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Beijing",
            "ST": "Beijing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes server-csr.json | cfssljson -bare server


# --------------------------------------------------
# 生成admin.pem admin-key.pem
# 用于集群管路员访问管理集群
# --------------------------------------------------
cat > admin-csr.json <<EOF
{
    "CN": "admin",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Beijing",
            "ST": "Beijing",
            "O": "system:master",
            "OU": "System"
        }
    ]
}
EOF

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes admin-csr.json | cfssljson -bare admin


# --------------------------------------------------
# 生成kube-proxy.pem kube-proxy-key-key.pem
# 用于集群管路员访问管理集群
# --------------------------------------------------
cat > kube-proxy-csr.json <<EOF
{
    "CN": "system:kube-proxy",
    "hosts": [],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Beijing",
            "ST": "Beijing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy

# --------------------------------------------------
# 生成aggregator-proxy.pem aggregator-proxy-key.pem
# 用于配置apiserver proxy-client参数
# --------------------------------------------------
cat > aggregator-proxy-csr.json <<EOF
{
    "CN": "system:aggregator-proxy",
    "hosts": [],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Beijing",
            "ST": "Beijing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes aggregator-proxy-csr.json | cfssljson -bare aggregator-proxy
```

```shell
wget https://github.com/etcd-io/etcd/releases/download/v3.5.6/etcd-v3.5.6-linux-amd64.tar.gz

wget https://cdn.dl.k8s.io/release/v1.26.8/kubernetes-node-linux-amd64.tar.gz
wget https://cdn.dl.k8s.io/release/v1.26.8/kubernetes-server-linux-amd64.tar.gz
wget https://cdn.dl.k8s.io/release/v1.26.8/kubernetes-client-linux-amd64.tar.gz
```

> master

### 单节点 etcd

```shell
mkdir -pv /opt/etcd/bin
tar -xvf etcd-v3.5.6-linux-amd64.tar.gz

cp -v etcd-v3.5.6-linux-amd64/etcd* /opt/etcd/bin/

mkdir /opt/ssl
cp -v ~/ssl/server.pem ~/ssl/server-key.pem /opt/ssl
cp -v ~/ssl/ca.pem ~/ssl/ca-key.pem /opt/ssl

cat <<EOF >> /usr/lib/systemd/system/etcd.service
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
ExecStart=/opt/etcd/bin/etcd \\
--advertise-client-urls=https://192.168.3.201:2379 \\
--client-cert-auth=true \\
--cert-file=/opt/ssl/server.pem \\
--key-file=/opt/ssl/server-key.pem \\
--peer-client-cert-auth=true \\
--peer-cert-file=/opt/ssl/server.pem \\
--peer-key-file=/opt/ssl/server-key.pem \\
--trusted-ca-file=/opt/ssl/ca.pem \\
--peer-trusted-ca-file=/opt/ssl/ca.pem \\
--data-dir=/var/lib/etcd \\
--experimental-initial-corrupt-check=true \\
--experimental-watch-progress-notify-interval=5s \\
--initial-advertise-peer-urls=https://192.168.3.201:2380 \\
--initial-cluster=k8s-master=https://192.168.3.201:2380 \\
--listen-client-urls=https://127.0.0.1:2379,https://192.168.3.201:2379 \\
--listen-metrics-urls=http://127.0.0.1:2381 \\
--listen-peer-urls=https://192.168.3.201:2380 \\
--name=k8s-master \\
--snapshot-count=10000
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable etcd
systemctl restart etcd
```

检查etcd集群状态

```
/opt/etcd/bin/etcdctl \
--cacert=/opt/ssl/ca.pem \
--cert=/opt/ssl/server.pem \
--key=/opt/ssl/server-key.pem \
--endpoints="https://192.168.3.201:2379" \
--write-out=table \
endpoint status 
```

### kube-apiserver

```shell
mkdir -pv /opt/kubernetes/bin

tar -xf kubernetes-server-linux-amd64.tar.gz
cd kubernetes/server/bin/

cp -v kubectl kube-apiserver kube-scheduler kube-controller-manager kube-proxy /opt/kubernetes/bin/

cp -v ~/ssl/aggregator-proxy.pem ~/ssl/aggregator-proxy-key.pem /opt/ssl
```

```bash
cat <<EOF >/usr/lib/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/opt/kubernetes/bin/kube-apiserver \\
--advertise-address=192.168.3.201 \\
--allow-privileged=true \\
--authorization-mode=Node,RBAC \\
--client-ca-file=/opt/ssl/ca.pem \\
--enable-admission-plugins=NodeRestriction \\
--enable-bootstrap-token-auth=true \\
--etcd-cafile=/opt/ssl/ca.pem \\
--etcd-certfile=/opt/ssl/server.pem \\
--etcd-keyfile=/opt/ssl/server-key.pem \\
--etcd-servers=https://127.0.0.1:2379 \\
--kubelet-client-certificate=/opt/ssl/server.pem \\
--kubelet-client-key=/opt/ssl/server-key.pem \\
--kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname \\
--proxy-client-cert-file=/opt/ssl/aggregator-proxy.pem \\
--proxy-client-key-file=/opt/ssl/aggregator-proxy-key.pem \\
--requestheader-allowed-names=front-proxy-client \\
--requestheader-client-ca-file=/opt/ssl/ca.pem \\
--requestheader-extra-headers-prefix=X-Remote-Extra- \\
--requestheader-group-headers=X-Remote-Group \\
--requestheader-username-headers=X-Remote-User \\
--secure-port=6443 \\
--service-account-issuer=https://kubernetes.default.svc.cluster.local \\
--service-account-key-file=/opt/ssl/ca.pem \\
--service-account-signing-key-file=/opt/ssl/ca-key.pem \\
--service-cluster-ip-range=10.96.0.0/12
--tls-cert-file=/opt/ssl/server.pem \\
--tls-private-key-file=/opt/ssl/server-key.pem
Restart=on-failure
RestartSec=5
Type=notify
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable kube-apiserver
systemctl restart kube-apiserver
```

```bash
# 生成 ~/.kube/config 配置文件
/opt/kubernetes/bin/kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=127.0.0.1:8443
/opt/kubernetes/bin/kubectl config set-credentials admin --client-certificate=admin.pem --embed-certs=true --client-key=admin-key.pem
/opt/kubernetes/bin/kubectl config set-context kubernetes --cluster=kubernetes --user=admin
/opt/kubernetes/bin/kubectl config use-context kubernetes

# kube-proxy.kubeconfig
/opt/kubernetes/bin/kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=127.0.0.1:8443 --kubeconfig=kube-proxy.kubeconfig
/opt/kubernetes/bin/kubectl config set-credentials kube-proxy --client-certificate=kube-proxy.pem --embed-certs=true --client-key=kube-proxy-key.pem --kubeconfig=kube-proxy.kubeconfig
/opt/kubernetes/bin/kubectl config set-context default --cluster=kubernetes --user=kube-proxy --kubeconfig=kube-proxy.kubeconfig
/opt/kubernetes/bin/kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig

# kube-controller-manager.kubeconfig
/opt/kubernetes/bin/kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=127.0.0.1:8443 --kubeconfig=kube-controller-manager.kubeconfig
/opt/kubernetes/bin/kubectl config set-credentials kube-controller-manager --client-certificate=kube-proxy.pem --embed-certs=true --client-key=kube-proxy-key.pem --kubeconfig=kube-controller-manager.kubeconfig
/opt/kubernetes/bin/kubectl config set-context default --cluster=kubernetes --user=kube-controller-manager --kubeconfig=kube-controller-manager.kubeconfig
/opt/kubernetes/bin/kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig

```

### kube-controller-manager

```bash
cat <<EOF > /usr/lib/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service]
ExecStart=/opt/kubernetes/bin/kube-controller-manager \\
--allocate-node-cidrs=true \\
--authentication-kubeconfig=/opt/kubernetes/etc/kube-controller-manager.kubeconfig \\
--authorization-kubeconfig=/opt/kubernetes/etc/kube-controller-manager.kubeconfig \\
--bind-address=127.0.0.1 \\
--client-ca-file=/opt/ssl/ca.pem \\
--cluster-cidr=10.244.0.0/16 \\
--cluster-name=kubernetes \\
--cluster-signing-cert-file=/opt/ssl/ca.pem \\
--cluster-signing-key-file=/opt/ssl/ca-key.pem \\
--controllers=*,bootstrapsigner,tokencleaner \\
--kubeconfig=/opt/kubernetes/etc/controller-manager.conf \\
--leader-elect=true \\
--requestheader-client-ca-file=/opt/ssl/ca.pem \\
--root-ca-file=/opt/ssl/ca.pem \\
--service-account-private-key-file=/opt/ssl/ca-key.pem \\
--service-cluster-ip-range=10.96.0.0/12 \\
--use-service-account-credentials=true
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl start kube-controller-manager
systemctl enable kube-controller-manager
```

### scheduler



> node

```shell
mkdir -pv /opt/kubernetes/bin

tar -xf kubernetes-node-linux-amd64.tar.gz
cd kubernetes/node/bin/

mv -v kubelet kube-proxy /opt/kubernetes/bin/
```

