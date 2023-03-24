[toc]

### ansible部署k8s

https://github.com/lizhenliang/ansible-install-k8s

#### 安装前准备

```shell
# 服务器时间同步

# 修改sshd DNS配置
sed -i 's/#UseDNS yes/UseDNS no/g' /etc/ssh/sshd_config
systemctl restart sshd

# ulimit限制
echo '* soft nofile 65535' >> /etc/security/limits.conf
echo '* hard nofile 65535' >> /etc/security/limits.conf
echo '* soft nproc 65535' >> /etc/security/limits.conf
echo '* hard nproc 65535' >> /etc/security/limits.conf
```

#### 安装ansible

```shell
# 随便选一台装ansible
curl https://mirrors.aliyun.com/repo/Centos-7.repo >> /etc/yum.repos.d/Centos-7.repo
curl http://mirrors.aliyun.com/repo/epel-7.repo >> /etc/yum.repos.d/epel-7.repo

yum install -y ansible

# 把 ansible-install-k8s.tar.gz binary_pkg.tar.gz 解压到 /root 下
```

#### 安装k8s集群

```shell
cd ansible-install-k8s

# README.md中有详细的安装步骤

# 修改 hosts, 注意lb-name不要修改, lb-name只能是lb-master和lb-backup

# 修改 group_vars/all.yml
```

#### 安装失败清理环境

```shell
# 在每台服务器执行这些命令, 执行完后, 重新跑ansible脚本

for i in docker kube-apiserver etcd kube-scheduler kube-controller-manager kubelet kube-proxy keepalived; do systemctl stop $i; systemctl disable $i;done

rm -rvf /tmp/k8s; rm -rvf /opt/*; rm -rvf /etc/keepalived/*; rm -rvf /var/lib/etcd
```

#### 安装kuboard-v3

```shell
kubectl apply -f https://addons.kuboard.cn/kuboard/kuboard-v3.yaml

# 选三台服务器, 打上etcd标签, kuboard会在在三个节点上创建etcd pod, 否则etcd无法启动; 如果只给一台服务器打etcd标签, 就只会创建一个etcd 
kubectl label nodes k8s-master01 k8s.kuboard.cn/role=etcd
kubectl label nodes k8s-master02 k8s.kuboard.cn/role=etcd
kubectl label nodes k8s-master03 k8s.kuboard.cn/role=etcd
```

修改docker存储目录

```shell
```

修改k8s nfs支持

