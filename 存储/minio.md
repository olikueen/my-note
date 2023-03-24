### minio

此方案适用 用户不提供专业存储的情况, 使用本地hostPath作为存储;

**创建minio数据目录**

```shell
mkdir -pv /data/minio
```

**给需要运行minio的节点打标签**

```shell
kubectl label nodes k8s-master01 minio-server=true
kubectl label nodes k8s-master02 minio-server=true
kubectl label nodes k8s-node01 minio-server=true
kubectl label nodes k8s-node02 minio-server=true
```

**安装minio**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: minio
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: minio
  namespace: minio
  labels:
    app: minio
spec:
  selector:
    matchLabels:
      app: minio
  template:
    metadata:
      labels:
        app: minio
    spec:
      nodeSelector:
        # 配置节点选择器, 只有在 minio-server=true 标签的节点上, 部署minio pod
        minio-server: "true"
      hostNetwork: true
      volumes:
      - name: storage
        hostPath:
          # 配置物理机minio存储路径
          path: /data/minio
      containers:
      - name: minio
        # minio console 的登录用户(MINIO_ACCESS_KEY)和密码(MINIO_SECRET_KEY)
        env:
        - name: MINIO_ACCESS_KEY
          value: "v9rwqYzXXim6KJKeyPm344"
        - name: MINIO_SECRET_KEY
          value: "0aIRBu9KU7gAN0luoX8uBE1eKWNPDgMnkVqbPC"
        image: minio/minio:RELEASE.2020-06-14T18-32-17Z
        # Unfortunately you must manually define each server. Perhaps autodiscovery via DNS can be implemented in the future.
        args:
        - server
        # 配置各个minio节点
        - http://k8s-master01/data/minio
        - http://k8s-master02/data/minio
        - http://k8s-node01/data/minio
        - http://k8s-node02/data/minio
        ports:
        - containerPort: 9000
        volumeMounts:
        - name: storage
          mountPath: /data/minio/
---
apiVersion: v1
kind: Service
metadata:
  name: minio
  namespace: minio
spec:
  type: NodePort
  ports:
  - name: web
    nodePort: 30900
    port: 9000
    protocol: TCP
    targetPort: 9000
  selector:
    app: minio
```

#### mc客户端使用方法

**下载mc客户端**

```shell
wget http://dl.minio.org.cn/client/mc/release/linux-amd64/mc
chmod 777 mc
mv mc /usr/local/bin
```

**添加 mc 配置文件, 两串随机字符串分别是 MINIO_ACCESS_KEY 和 MINIO_SECRET_KEY**

```shell
# mc config host add minio节点别名 minio节点url MINIO_ACCESS_KEY MINIO_SECRET_KEY --api s3v4
mc config host add minio http://192.168.101.141:9000 v9rwqYzXXim6KJKeyPm344 0aIRBu9KU7gAN0luoX8uBE1eKWNPDgMnkVqbPC --api s3v4
# 配置文件存储在 ~/.mc 中, 查看配置
mc config host list
```

**创建存储桶**

```shell
# mc mb minio节点别名/存储桶名
[root@k8s-master01 aa]# mc mb minio/test2
Bucket created successfully `minio/test2`.

# mc mb minio节点别名/存储桶名/文件路径
[root@k8s-master01 aa]# mc mb minio/test3/a/b/c
Bucket created successfully `minio/test3/a/b/c`.
```

查看minio中存储桶

```shell
# mc ls minio节点别名
[root@k8s-master01 aa]# mc ls minio
[2021-08-20 13:52:02 CST]     0B pvc-f84801b2-c9e2-4a06-a257-e40eb0e8e0de/
[2021-08-19 15:51:36 CST]     0B test/
[2021-08-20 17:00:31 CST]     0B test2/
[2021-08-20 17:02:11 CST]     0B test3/

[root@k8s-master01 aa]# mc ls minio/test3
[2021-08-20 17:02:59 CST]     0B a/
```

上传文件

```shell
[root@k8s-master01 aa]# date > testfile.txt

[root@k8s-master01 aa]# mc cp testfile.txt minio/test3/a/
testfile.txt:                     43 B / 43 B ┃▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓┃ 1.58 KiB/s 0s 

[root@k8s-master01 aa]# mc ls minio/test3/a
[2021-08-20 17:05:18 CST]    43B testfile.txt
[2021-08-20 17:06:51 CST]     0B b/
```

删除文件

```shell
[root@k8s-master01 aa]# mc rm minio/test3/a/testfile.txt
Removing `minio/test3/a/testfile.txt`.
```

#### 权限设置

要想通过url直接下载minio中的文件, 需要给minio的桶配置权限

权限默认有 none, download, upload, public 种, 配置相应的权限, 可以开启对应的 免认证访问;

```shell
# 我预先在test桶中上传了一个文件
[root@k8s-master01 ~]# mc cp kuboard-v3.yaml minio/test

# 查看权限
[root@k8s-master01 tmp]# mc policy get minio/test
Access permission for `minio/test` is `none`
# 这种时候, 无论上传还是下载, 都需要先通过权限验证才行

# 设置权限, 给minio的test桶开启download权限
[root@k8s-master01 ~]# mc policy set download minio/test
Access permission for `minio/test` is set to `download`

# 这时候就可以免密下载文件了 http://192.168.101.141:9000/minio/download/<存储桶>/<文件路径>?token=
[root@k8s-master01 tmp]# wget http://192.168.101.141:9000/minio/download/test/kuboard-v3.yaml?token=
```

