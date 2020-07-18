# 说明



安装启动minikube可以选择允许在虚拟机上也可以选择运行在宿主机上

通过minikube start 时 设置参数 --vm-driver 选择运行方式   node为在宿主机上运行
选择在虚拟机上运行时需要系统开启虚拟化，而选择在宿主机上运行时操作系统只能是linux 且必须安装docker



# docker安装

#### 移除旧版本


                 sudo yum remove docker \
                	docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine

此时仍有可能没有移除干净  
可以通过 yum list installed|grep docker查看docker是否还存在

#### 安装系统必要的工具

sudo yum install -y yum-utils device-mapper-persistent-data lvm2

#### 添加软件源信息

sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#### 查看可用版本的docker-ce

yum list docker-ce --showduplicates | sort -r

#### 安装指定版本的docker

sudo yum install -y docker-ce-17.12.1.ce-1.el7.centos 

安装完docker后记得先启动在查看安装版本号
否则docker server 将显示不能连接

这里选择17.12.1版本docker安装，因为之后安装的minikube如果选择最新版的则需要多核cpu
而安装v0.30.0的minikube的话需要低版本的docker兼容

# kubectl安装

下载kubectl 这个客户端连接工具可以选择最新版

```shell
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

更改文件权限同时添加到环境中


kubectl version
Client Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.0", GitCommit:"70132b0f130acc0bed193d9ba59dd186f0e634cf", GitTreeState:"clean", BuildDate:"2019-12-07T21:20:10Z", GoVersion:"go1.13.4", Compiler:"gc", Platform:"linux/amd64"}
The connection to the server 192.168.0.63：8843 was refused - did you specify the right host or port?

minikube安装

```shell
curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v0.30.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

可以选择高版本安装但CPU不能是单核的否则将直接启动不了

minikube version
minikube version: v0.25.0



minikube start --registry-mirror=https://registry.docker-cn.com --vm-driver=none

kubectl cluster-info
Kubernetes master is running at https://192.168.0.63:8443
CoreDNS is running at https://192.168.0.63:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy