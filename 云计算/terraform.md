## 1 安装

```shell
wget https://releases.hashicorp.com/terraform/1.6.6/terraform_1.6.6_linux_amd64.zip

unzip terraform_1.6.6_linux_amd64.zip

mv terraform /usr/sbin/
```

安装docker(学习环境)

```shell
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -P /etc/yum.repos.d/

yum install -y docker-ce

systemctl enable docker

mkdir -p /etc/docker
tee /etc/docker/daemon.json << 'EOF'
{
"registry-mirrors": ["https://rfj1yucr.mirror.aliyuncs.com"]
}
EOF

systemctl daemon-reload
systemctl restart docker
```

设置代理

```shell
export https_proxy=http://192.168.3.1:7890
export no_proxy=172.17.*.*,127.0.0.1,192.168.3.*
```

## 2 部署一个nginx

```bash
mkdir learn-terraform-docker-container
cd learn-terraform-docker-container

vim main.tf
```



```shell
# 初始化环境
terraform init

# 部署资源
terraform apply

# 销毁资源
terraform destroy
```

