[toc]
# 基本集群搭建
## 环境说明
```
3台服务器：
    2核心 2G内存 100G硬盘 无swap分区

部署架构：
    1台Master节点： kube-scheduler, kube-apiserver, kube-controller-manager, etcd
    2台Node节点：   kubelet, kubeproxy, docker

Docker: docker-ce-19.03.7
Kubernetes: 1.20.7
```

## 服务器配置
### 安装前准备
```
# 关闭防火墙：
systemctl stop firewalld
systemctl disable firewalld

# 关闭selinux：
sed -i '/^SELINUX=/ s/enforcing/disabled/' /etc/selinux/config
setenforce 0

# 关闭swap：
swapoff -a $ 临时
vim /etc/fstab $ 永久

# 添加主机名与IP对应关系（记得设置主机名）：
cat <<EOF >> /etc/hosts
192.168.88.139 k8s-master01 k8s-master01.alec.com
192.168.88.140 k8s-node01 k8s-node01.alec.com
192.168.88.141 k8s-node02 k8s-node02.alec.com
EOF

# 将桥接的IPv4流量传递到iptables的链：
cat <<EOF >> /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
vm.swappiness=0
fs.file-max=52706963
fs.nr_open=52706963
EOF

# 应用内核参数
sysctl -p

时间同步：
......

修改时区:

```

### 安装yum源
```
curl https://mirrors.aliyun.com/repo/Centos-7.repo > /etc/yum.repos.d/Centos-7.repo
curl https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo > /etc/yum.repos.d/docker-ce.repo

cat <<EOF > /etc/yum.repos.d/kubernetes.repo 
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

rpm --import https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
```

### 在所有服务器安装docker
```
yum -y install docker-ce-20.10.16

# 配置docker加速器
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://rfj1yucr.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

systemctl daemon-reload
systemctl enable docker
systemctl restart docker
```

## Master节点安装
### 安装
```
yum install -y kubelet-1.22.11 kubeadm-1.22.11 kubectl-1.22.11
systemctl enable kubelet
```
```
# 初始化Master节点
kubeadm init \
--image-repository registry.aliyuncs.com/google_containers \
--apiserver-advertise-address=192.168.91.181 \
--kubernetes-version=v1.22.11 \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16 \
--v=5
```
```shell
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.88.139:6443 --token 3c919y.tq0z4s2l1iw1x0p3 \
    --discovery-token-ca-cert-hash sha256:5680a5278c2ce7afd3466b1b1de5458268a25b9c8cef2d442a55de363f16565a
```
> 普通用户

```
useradd k8s
echo "k8s ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

增加配置文件：
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

[k8s@k8s-master01 ~]$ kubectl get nodes
NAME           STATUS     ROLES                  AGE     VERSION
k8s-master01   NotReady   control-plane,master   7m37s   v1.20.7
```
> root用户

```
[root@k8s-master01 ~]# echo 'export KUBECONFIG=/etc/kubernetes/admin.conf' >> /etc/profile
[root@k8s-master01 ~]# source /etc/profile
[root@k8s-master01 ~]# kubectl get nodes
NAME           STATUS     ROLES                  AGE     VERSION
k8s-master01   NotReady   control-plane,master   8m10s   v1.20.7
```

### 安装网络驱动

> ~~安装flannel网络驱动(已淘汰)~~

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml
```

> 安装calico网络驱动

```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```



<hr>

```
查看node状态：
[root@master ~]# kubectl get nodes
NAME              STATUS     ROLES    AGE    VERSION
master.alec.com   NotReady   master   101m   v1.13.3
node01.alec.com   NotReady   <none>   100m   v1.13.3
node02.alec.com   NotReady   <none>   100m   v1.13.3

显示都是NotReady是由于flannel未安装；

查看flannel pod状态
[root@master ~]# kubectl get pods -n kube-system
NAME                                      READY   STATUS                  RESTARTS   AGE
coredns-78d4cf999f-qxb6z                  0/1     Pending                 0          3h27m
coredns-78d4cf999f-s8gqd                  0/1     Pending                 0          3h27m
etcd-master.alec.com                      1/1     Running                 0          3h26m
kube-apiserver-master.alec.com            1/1     Running                 0          3h26m
kube-controller-manager-master.alec.com   1/1     Running                 0          3h26m
kube-flannel-ds-amd64-ffhn9               0/1     Init:ImagePullBackOff   0          14m
kube-proxy-7tg8j                          1/1     Running                 0          3h27m
kube-scheduler-master.alec.com            1/1     Running                 0          3h26m

flannel状态为Init:ImagePullBackOff ，kubectl get node 显示为 NotReady；

解决办法：手动pull flannel镜像
docker pull quay.io/coreos/flannel:v0.11.0-amd64
从官方的点可能拉取不成功，从国内的站点拉：
docker pull quay-mirror.qiniu.com/coreos/flannel:v0.11.0-amd64
拉取后修改tag和官方一致：
docker tag quay-mirror.qiniu.com/coreos/flannel:v0.11.0-amd64 quay.io/coreos/flannel:v0.11.0-amd64
再次查看pods状态，已更新为Running，node状态更新为Ready；
```


## Node节点安装
### 安装
```
yum install -y kubelet-1.22.11 kubeadm-1.22.11
systemctl enable kubelet
```
### 添加Node到集群
```
从init的输出信息中获取加入命令，执行：
kubeadm join 192.168.88.139:6443 --token 3c919y.tq0z4s2l1iw1x0p3 \
    --discovery-token-ca-cert-hash sha256:5680a5278c2ce7afd3466b1b1de5458268a25b9c8cef2d442a55de363f16565a
```
```
从Master查看Node节点：
[root@master ~]# kubectl get node
NAME              STATUS     ROLES    AGE     VERSION
master.alec.com   Ready      master   4h28m   v1.13.3
node01.alec.com   NotReady   <none>   50s     v1.13.3

等待一会，node节点会从master节点拉取calico镜像，之后就会更新为Ready;
```

### 添加Node到集群2

kubeadm 默认token有效期为24h, 超过24h后再往集群中添加Node节点, 需要重新生成token;

```shell
# 安装docker kubeadm kubelet
# 生成新的token
[root@t-c7u9-k8smaster01 ~]# kubeadm token create
zxd0s3.efgbvwz7p7ttgsl5
# 查看token
[root@t-c7u9-k8smaster01 ~]# kubeadm token list
TOKEN                     TTL         EXPIRES                     USAGES                   DESCRIPTION                                                EXTRA GROUPS
zxd0s3.efgbvwz7p7ttgsl5   23h         2021-07-28T15:11:01+08:00   authentication,signing   <none>                                                     system:bootstrappers:kubeadm:default-node-token

# 获取ca证书sha256编码hash值
[root@t-c7u9-k8smaster01 ~]# openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
55024eadd986a38ccbd4eeaaafda9e710a6b3c56a0dd0ca73d6b61cc8f2673ae

# 节点加入集群
[root@t-c7u9-k8snode02 ~]# kubeadm join 192.168.101.131:6443 --token zxd0s3.efgbvwz7p7ttgsl5 \
--discovery-token-ca-cert-hash sha256:55024eadd986a38ccbd4eeaaafda9e710a6b3c56a0dd0ca73d6b61cc8f2673ae \
--v=5
```

> 脚本: 生成加入集群命令

```shell
#!/bin/bash
APISERVER="192.168.101.131:6443"
TOKEN=$(kubeadm token create)
HASH=$(openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //')
echo "kubeadm join $APISERVER --token $TOKEN --discovery-token-ca-cert-hash sha256:$HASH --v=5"
```



## 测试k8s集群

```
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl get pod,svc -o wide
```
浏览器访问http://任一节点IP:PORT



## ~~部署DashBoard~~

默认kubernetes-dashboard.yaml中的镜像地址在国外，国内获取不到；<br>
默认Dashboard只能集群内部访问，修改Service为NodePort类型，暴露到外部；<br>

- 可以按照下面的方式部署；
- 也可以先搭建本地docker仓库，然后修改kubernetes-dashboard.yaml中"image: k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1"为本地仓库路径；

```
在所有节点，从阿里云把镜像拉取到本地：
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kubernetes-dashboard-amd64:v1.10.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kubernetes-dashboard-amd64:v1.10.1 k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1

获取kubernetes-dashboard.yaml文件
wget https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml

修改kubernetes-dashboard.yaml，在Service中增加type: NodePort，将pod端口暴露出来；
    kind: Service
    apiVersion: v1
    metadata:
    labels:
        k8s-app: kubernetes-dashboard
    name: kubernetes-dashboard
    namespace: kube-system
    spec:
    type: NodePort
    ports:
        - port: 443
        targetPort: 8443
    selector:
        k8s-app: kubernetes-dashboard

再执行 kubectl apply -f kubernetes-dashboard.yaml 生成pod；
```

```
检查：
[root@master ~]# kubectl get pod -n kube-system
NAME                                      READY   STATUS    RESTARTS   AGE
......
kubernetes-dashboard-57df4db6b-np4gl      1/1     Running   0          11m

[root@master ~]# kubectl get svc -n kube-system
NAME                   TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
......
kubernetes-dashboard   NodePort    10.1.23.57   <none>        443:32692/TCP   5m21s

浏览器访问http://任一节点IP:PORT
```

谷歌浏览器不能访问的问题：
```
[root@master ~]# mkdir key && cd key

# 生成证书
[root@master key]# openssl genrsa -out dashboard.key 2048 
Generating RSA private key, 2048 bit long modulus
.....................+++
...............................................+++
e is 65537 (0x10001)
[root@master key]# openssl req -new -out dashboard.csr -key dashboard.key -subj '/CN=192.168.30.130'
[root@master key]# openssl x509 -req -in dashboard.csr -signkey dashboard.key -out dashboard.crt 
Signature ok
subject=/CN=192.168.30.130
Getting Private key

#删除原有的证书secret
[root@master key]# kubectl delete secret kubernetes-dashboard-certs -n kube-system

# 创建新的证书secret
[root@master key]# kubectl create secret generic kubernetes-dashboard-certs --from-file=dashboard.key --from-file=dashboard.crt -n kube-system

# 查看pod
[root@master key]# kubectl get pod -n kube-system
NAME                                      READY   STATUS    RESTARTS   AGE
......
kubernetes-dashboard-57df4db6b-np4gl      1/1     Running   0          21m

# 重启pod
[root@master key]# kubectl delete pod kubernetes-dashboard-57df4db6b-np4gl -n kube-system
```

创建service account并绑定默认cluster-admin管理员集群角色(给dashbord生成令牌)：
```
[root@master key]# kubectl create serviceaccount dashboard-admin -n kube-system
[root@master key]# kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin

查看secret：
[root@master key]# kubectl -n kube-system get secret
NAME                                             TYPE                                  DATA   AGE
......
dashboard-admin-token-9f6xv                      kubernetes.io/service-account-token   3      2m1s
......

查看token：
[root@master key]# kubectl describe secrets -n kube-system dashboard-admin-token-9f6xv
```



## 安装helm

- **下载** https://github.com/helm/helm/releases

- **安装**

  ```shell
  tar -xf helm-v3.9.0-linux-amd64.tar.gz
  cd linux-amd64
  mv helm /usr/local/bin/
  ```


## 安装ingress-nginx

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nginx-ingress

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress 
  namespace: nginx-ingress

---
apiVersion: v1
kind: Secret
metadata:
  name: default-server-secret
  namespace: nginx-ingress
type: Opaque
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURWekNDQWorZ0F3SUJBZ0lKQUtaRTllUmZaRlJSTUEwR0NTcUdTSWIzRFFFQkN3VUFNRUl4SHpBZEJnTlYKQkFNTUZrNUhTVTVZU1c1bmNtVnpjME52Ym5SeWIyeHNaWEl4SHpBZEJnTlZCQW9NRms1SFNVNVlTVzVuY21WegpjME52Ym5SeWIyeHNaWEl3SGhjTk1qRXdNVEkxTURjeE9USTRXaGNOTXpFd01USXpNRGN4T1RJNFdqQkNNUjh3CkhRWURWUVFEREJaT1IwbE9XRWx1WjNKbGMzTkRiMjUwY205c2JHVnlNUjh3SFFZRFZRUUtEQlpPUjBsT1dFbHUKWjNKbGMzTkRiMjUwY205c2JHVnlNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQQp0M1hVcFhYcUdDUTJBblRTd21DMjZ1bzRudGhmQWNYd3U4REdLUVlBSGQ1VlFKaDRPdStSWEJ4M1BTMVlTY3VNCmVTVXExV1R1ejQzRW9tRHZOWkdHcmNkSlJrWDhjYk05WWNBanlWZ0pnd0cxVFB0OUl2ZzVibWJBWE1INnR3OTUKRGJiOFFlSWVSeXBUWVl6bXJmMnJxaU45aXdROUMzSHhySDc3ektoZ1hxT20vZ09CdGpWRVhKc3lSeEpTcUdJZQpiNUp6NENJK0FzaUZGYWRjMHZhcmpvei8wTTJuS1JaU3VUQThmRHhsMUpTZlVJc2hYWUt1YmZvMzh0ck1paHFoCnh2Q09SMWFJUmxPaDNDTXAyekRoUm9nNHpjTFdyUHBXQnlyVkN2aGVBWWp2MitpWHlTQ1MyU2pRbmV4UE43aWoKUE5Kc0hpZVZMdVE0R0VCYTJ3RkxFUUlEQVFBQm8xQXdUakFkQmdOVkhRNEVGZ1FVSllDdmVtZFpFQVM4eGIxUAo3bVo3YjBGRFViZ3dId1lEVlIwakJCZ3dGb0FVSllDdmVtZFpFQVM4eGIxUDdtWjdiMEZEVWJnd0RBWURWUjBUCkJBVXdBd0VCL3pBTkJna3Foa2lHOXcwQkFRc0ZBQU9DQVFFQW1Nc2QwbVRtaFBXc2pvejh0NUN2dTdXMVNhT2EKTzRQTlRNakJ3eEpHZUt1ZUlGNytSZWR3WUVaWXJ0M1hiY0NHRDAxMklFMjNJK1k0eWs4ekFkdWxtMmtZRUdPZQp6UDQyRGpCcFpZelhiUmpkQkY0aVFlMWR6K0xVWGJyK096bWFoRjMwOW5MVU5KVStNTStHbVVUbmlwdy90dGxnCkszNlI3WWpuSzNpelhqaE8rdFJvWnp5eWhsdDRVcmQxYTN0Tkxacy8yNk14akZ2aGlOWUNNSmh4blMzUlVzWXYKdllxUUw0WkQ1b0VaTVIvUEN3VlR3OHpTUEVuU0hlMnFLT2dmZGlOYTEwTHpYYzc0eGNmK2FZTmlvcndNdEpXdwo5NnR1WExUN3ZNMW11Ky9oM3BCR0RaMFY3SitwZTlvV2daZSsrNVVINyswK2tpSlN5S1VCN29LMHN3PT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQ==
  tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV2Z0lCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQktnd2dnU2tBZ0VBQW9JQkFRQzNkZFNsZGVvWUpEWUMKZE5MQ1lMYnE2amllMkY4QnhmQzd3TVlwQmdBZDNsVkFtSGc2NzVGY0hIYzlMVmhKeTR4NUpTclZaTzdQamNTaQpZTzgxa1lhdHgwbEdSZnh4c3oxaHdDUEpXQW1EQWJWTSszMGkrRGx1WnNCY3dmcTNEM2tOdHZ4QjRoNUhLbE5oCmpPYXQvYXVxSTMyTEJEMExjZkdzZnZ2TXFHQmVvNmIrQTRHMk5VUmNtekpIRWxLb1loNXZrblBnSWo0Q3lJVVYKcDF6UzlxdU9qUC9RemFjcEZsSzVNRHg4UEdYVWxKOVFpeUZkZ3E1dCtqZnkyc3lLR3FIRzhJNUhWb2hHVTZIYwpJeW5iTU9GR2lEak53dGFzK2xZSEt0VUsrRjRCaU8vYjZKZkpJSkxaS05DZDdFODN1S004MG13ZUo1VXU1RGdZClFGcmJBVXNSQWdNQkFBRUNnZ0VBQlAvbkRhTksvK0ZzdjJCanBmeHd2N0ltWE4zVXFQMjE4OGZySG84VlRic1QKWTdGRUJZY2wxUGJKb1JjdFFzV1RUSEhnMnZQbk5pek00UWYzUE9SOFlSdi9PVFVMRGlZdVZBMmliQWhFS2hmUAowd3MvZThaNytqQStxY2gzaHFtYlNPNWxyWDMyQ1VaMEEwS052c3djODRRSUZkUEZ2aHdhMC9LWjloZlltSHVkCmFOcGcvbzJuS3dSUVJSYjZYVkpsWWNjTHcvMmE1VzdQMGM0NENWWnI0VDh5MEErWHBUaTF3YUh6VlloQTZOc1QKemE1emJBS3ljMjZ2NWRhQ3VCT1JhYjkzWTFIUFRjOG9VRGIwR1VQa3BNd2dOaW9TV01rakNHRzBWWGtRUFE4NwplWlVteDNrbXNCVDF6RGtPR3pnNm9oSmlpSVQvcFhoVWVIckc1K0doWVFLQmdRRG5vUE0xQmNkeFArTGpMWDVrCnJqamxuSzA3SlQ3N0RlZ0o4VmtUQU90VDQ2cmJVUGxTT0VQTEV4ZUF6dnBkM3VWNkYwa1dYcDI0L0oydlZrSXcKTklQOWJxZ0RQYzBldFdhT0hLMHloanRuM1N1VTJreGtaQTBGN3lHNEFnTzU5SlZidWNnYWpVOStKcDc0TEdHTQprVmUrc09WMGlNVDRmWGMyQzJLUWJtNlduUUtCZ1FES3cyL0lyM25ucUI0dU44bFVSdUZ6MnBNQytTK3RLZlFVCi8xSEFSODAvRDZOYU5xdGhKcXFLd015Z091OGVjcTBMVUhFdm43T1ZyTk9aYzBHcktFQzJnTjF6YXRXVURHcWYKY3J5NnovRjVVQVkwOTI5K004bUJMeWFKeldudWtYaUUwcURlQ2p0cDh4Y3hiZnJOZ1JBU1h3NytudW1jb0RxUAoxb0dSMnQyaUJRS0JnUUNRS1U5Vlg5eHFzdDF1a1VFS1Bwanc1NXUxcFEvV3h5ZjFFRDVsSW54VXdPejFCU2UzCnNZY1lIRERUblg2YjcvK1pCbWNad2hlZUs3T2tqaVl4eEcybHpUcEtraXRaQW9QcXpSUkt6dHFvWVRJZnVlSXoKMVVWNXZRU2FkcjZFL1NIOGJkdUtFd3MzczZmYlJCd09sZU1ycndPUWpSTXlxVHdKNmZvVmRIWGx6UUtCZ0RCUwp3WnBmajdzUkN4aFN2VTJ6a3Rtc2x1clhmbkJUbGxOR3dqSUVLcnREdTlldFBjenFqU3lDWklJdmFYdWxNdTZHClhtTk9PVnVMaytaM1hJZ3hFTE11SlJqenRqRVJnSHU5dVpNQUtmbVNnOWd0dkVta2gvcWN4UitFY0NHbVU4VzcKK1JEUitYVDN0V2hYWUxXSGM5QWREWkxMUnJ2SVNBeXR2N1dHSnRvTkFvR0JBSlJPMzhxOVhuRHZBeEVRUzdpWgphd2dOWFA2TnBiSnVIeUo0cDJ4VldIV0N4VDhrL0E3eDVrb0FrTEpudTUvRE10d0pVL09ZRUpPY1k1b0l2UUVsCnAybWkxM3dEcVpvNml4d2kvU3h5T2xRQ2ZmNHBjNTlTUzE1WVM1RGVEeVVuakcwNTNha2VyQ3R5TGQ4Z2xnOUYKSTE2Y05TQXdTbDFvVi9KM21qOGdzZHpmCi0tLS0tRU5EIFBSSVZBVEUgS0VZLS0tLS0=
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-config
  namespace: nginx-ingress
data:
  client-max-body-size: "0"
  server-names-hash-bucket-size: "1024"


---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nginx-ingress
rules:
- apiGroups:
  - ""
  resources:
  - services
  - endpoints
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - secrets
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - configmaps
  verbs:
  - get
  - list
  - watch
  - update
  - create
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - list
- apiGroups:
  - ""
  resources:
  - events
  verbs:
  - create
  - patch
- apiGroups:
  - extensions
  resources:
  - ingresses
  verbs:
  - list
  - watch
  - get
- apiGroups:
  - "extensions"
  resources:
  - ingresses/status
  verbs:
  - update
- apiGroups:
  - k8s.nginx.org
  resources:
  - virtualservers
  - virtualserverroutes
  verbs:
  - list
  - watch
  - get

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nginx-ingress
subjects:
- kind: ServiceAccount
  name: nginx-ingress
  namespace: nginx-ingress
roleRef:
  kind: ClusterRole
  name: nginx-ingress
  apiGroup: rbac.authorization.k8s.io

---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-ingress
  namespace: nginx-ingress
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "9113"
spec:
  selector:
    matchLabels:
      app: nginx-ingress
  template:
    metadata:
      labels:
        app: nginx-ingress
    spec:
      serviceAccountName: nginx-ingress
      containers:
      - image: nginx/nginx-ingress:1.7.2
        imagePullPolicy: IfNotPresent
        name: nginx-ingress
        ports:
        - name: http
          containerPort: 80
          hostPort: 80
        - name: https
          containerPort: 443
          hostPort: 443
        - name: prometheus
          containerPort: 9113
        env:
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        args:
          - -nginx-configmaps=$(POD_NAMESPACE)/nginx-config
          - -default-server-tls-secret=$(POD_NAMESPACE)/default-server-secret
          - -enable-prometheus-metrics
```

```shell
kubectl apply -f ingress-nginx.yaml
```



## 安装kubesphere

```shell
```





# 日常操作

## k8s删除node节点

```shell
# 驱逐节点上的pod
kubectl drain k8s-node03 --delete-local-data --ignore-daemonsets
# 删除node节点
kubectl delete node k8s-node03
```



