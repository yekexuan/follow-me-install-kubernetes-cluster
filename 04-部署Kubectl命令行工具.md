<!-- toc -->

tags: kubectl

# 部署 kubectl 命令行工具

本文档介绍下载和配置 kubernetes 集群命令行工具 kubectl 的步骤。

kublet 默认使用 `localhost:8080` 访问 kube-apiserver，而通常情况下执行 kublet 命令的机器并不是 kube-apiserver，故执行时出错：

``` bash
$  kubectl get pods
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

故需要为 kublet 生成 `~/.kube/config` 配置文件， 然后拷贝到**所有使用 kublet 命令的机器**。

## 使用的变量

本文档用到的变量定义如下：

``` bash
$ export MASTER_IP=10.64.3.7 # 替换为 kubernetes maste 集群任一机器 IP
$ export KUBE_APISERVER="https://${MASTER_IP}:6443"
$
```

+ 变量 KUBE_APISERVER 指定 kublet 访问的 kube-apiserver 的地址，后续被写入 `~/.kube/config` 配置文件。

## 下载 kubectl

``` bash
$ wget https://dl.k8s.io/v1.6.1/kubernetes-client-linux-amd64.tar.gz
$ tar -xzvf kubernetes-client-linux-amd64.tar.gz
$ sudo cp kubernetes/client/bin/kube* /root/local/bin/
$ chmod a+x /root/local/bin/kube*
$ export PATH=/root/local/bin:$PATH
$
```

## 创建 kubectl kubeconfig 文件

``` bash
$ # 设置集群参数
$ kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER}
$ # 设置客户端认证参数
$ kubectl config set-credentials admin \
  --client-certificate=/etc/kubernetes/ssl/admin.pem \
  --embed-certs=true \
  --client-key=/etc/kubernetes/ssl/admin-key.pem
$ # 设置上下文参数
$ kubectl config set-context kubernetes \
  --cluster=kubernetes \
  --user=admin
$ # 设置默认上下文
$ kubectl config use-context kubernetes
```

+ `admin.pem` 证书 OU 字段值为 `system:masters`，`kube-apiserver` 预定义的 RoleBinding `cluster-admin` 将 Group `system:masters` 与 Role `cluster-admin` 绑定，该 Role 授予了调用`kube-apiserver` 相关 API 的权限；
+ 生成的 kubeconfig 被保存到 `~/.kube/config` 文件；

## 分发 kubeconfig 文件

将 `~/.kube/config` 文件拷贝到运行 `kubelet` 命令的机器的 `~/.kube/` 目录下。