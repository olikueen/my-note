### 1 docker启动参数

> 新版本的kubernetes要求docker增加"native.cgroupdriver=systemd"启动参数

```
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
"registry-mirrors": ["https://rfj1yucr.mirror.aliyuncs.com"],
"exec-opts":["native.cgroupdriver=systemd"]
}
EOF

systemctl daemon-reload
systemctl restart docker
```



### 2 cri-dockerd

在kubernetes 1.24 版本中, 移除了dockerism, kubelet 不再兼容 docker;

需要手动安装 cri-dockerd, 以连结 kubelet 和 docker;
