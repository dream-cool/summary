#why k8s?

###部署应用的历程

 传统部署时代   -->   虚拟化部署时代    -->    容器化部署时代

###容器化的好处

**敏捷地创建和部署应用程序**

**微服务使得应用程序更为分散**

**分离开发和运维的关注点**

**可监控性**

**开发、测试、生产不同阶段的环境一致性**

**跨云服务商、跨操作系统发行版的可移植性**

**资源隔离、资源利用**



###k8s的功能

**服务发现和负载均衡、存储编排、自动发布和回滚、自愈、密钥及配置管理**



### k8s对象

Kubernetes对象指的是Kubernetes系统的持久化实体，Kubernetes将其数据以Kubernetes对象的形式通过 api server存储在 etcd 中。一个Kubernetes对象代表着用户的一个意图（a record of intent），一旦您创建了一个Kubernetes对象，Kubernetes将持续工作，以尽量实现此用户的意图

spec：描述了您对该对象所期望的 **目标状态**

status： 只能由 Kubernetes 系统来修改，描述了该对象在 Kubernetes 系统中的 **实际状态**

Kubernetes 系统将不断地比较 **实际状态** staus 和 **目标状态** spec 之间的差异，并根据差异做出对应的调整。



根据.yaml文件生成k8s对象时，如下字段必填：

- **apiVersion** 用来创建对象时所使用的Kubernetes API版本
- **kind** 被创建对象的类型
- **metadata** 用于唯一确定该对象的元数据：包括 `name` 和 `namespace`，如果 `namespace` 为空，则默认值为 `default`
- **spec** 描述您对该对象的期望状态



### names

Kubernetes REST API 中，所有的对象都是通过 `name` 和 `UID` 唯一性确定。

Kubernetes集群中，每创建一个对象，都有一个唯一的 UID。用于区分多次创建的同名对象（如前所述，按照名字删除对象后，重新再创建同名对象时，两次创建的对象 name 相同，但是 UID 不同。）

可以通过 `namespace` + `name` 唯一性地确定一个 RESTFUL 对象，例如`/api/v1/namespaces/{namespace}/pods/{name}`，即k8s对象中的selfLink



### namespace

名称空间为 [名称](https://kuboard.cn/learning/k8s-intermediate/obj/names.html) 提供了作用域。名称空间内部的同类型对象不能重名，但是跨名称空间可以有同名同类型对象。名称空间不可以嵌套，任何一个Kubernetes对象只能在一个名称空间中。

名称空间可以用来在不同的团队（用户）之间划分集群的资源。

nodes、persistentVolumes、storageClass 不在任何名称空间。



### label

使用标签，用户可以按照自己期望的形式组织 Kubernetes 对象之间的结构，而无需对 Kubernetes 有任何修改。

基于等式的选择方式和基于集合的选择方式。

```
selector:
  matchLabels:
    component: redis
  matchExpressions:
    - {key: tier, operator: In, values: [cache]}
    - {key: environment, operator: NotIn, values: [dev]}
```



### RuntimeClass

通过 RuntimeClass，使不同的 Pod 使用不同的容器引擎。



### 容器

postStart 、 preStop 钩子

```
apiVersion: v1
kind: Pod
metadata:
  name: lifecycle-demo
spec:
  containers:
  - name: lifecycle-demo-container
    image: nginx
    lifecycle:
      postStart:
        exec:
          command: ["/bin/sh", "-c", "echo Hello from the postStart  > /usr/share/message"]
      preStop:
        exec:
          command: ["/bin/sh","-c","nginx -s quit; while killall -0 nginx; do sleep 1; done"]
```



### Pod

- 同一个 Pod 中的所有容器 IP 地址都相同
- 同一个 Pod 中的不同容器不能使用相同的端口，否则会导致端口冲突
- 同一个 Pod 中的不同容器可以通过 localhost:port 进行通信
- 同一个 Pod 中的不同容器可以通过使用常规的进程间通信手段，例如 SystemV semaphores 或者 POSIX 共享内存

应该始终通过创建 Controller 来创建 Pod，而不是直接创建 Pod。

### Deployment

#####创建

pod-template-hash 标签是 Deployment 创建 ReplicaSet 时添加到 ReplicaSet 上的，ReplicaSet 进而将此标签添加到 Pod 上。这个标签用于区分 Deployment 中哪个 ReplicaSet 创建了哪些 Pod。该标签的值是 `.spec.template` 的 hash 值。



#####更新

当 Deployment 的 rollout 正在进行中的时候，如果您再次更新 Deployment 的信息，此时 Deployment 将再创建一个新的 ReplicaSet 并开始将其 scale up，将先前正在 scale up 的 ReplicaSet 也作为一个旧的 ReplicaSet，并开始将其 scale down。



#####回滚

可以设定 revision history limit 来确定保存的历史版本数量。

kubectl rollout history  deployment/deployment-name   查看 Deployment 的历史版本

kubectl rollout history  deployment/deployment-name  -revision=2  查看 revision的详细信息

kubectl rollout undo deployment/deployment-name 回滚到前一个版本

kubectl rollout undo deployment/deployment-name   -revision=2  回滚到指定版本

不能回滚（rollback）一个已暂停的 Deployment，除非您继续（resume）该 Deployment。



#####伸缩



#####暂停和继续

您可以先暂停 Deployment，然后再触发一个或多个更新，最后再继续（resume）该 Deployment。这种做法使得您可以在暂停和继续中间对 Deployment 做多次更新，而无需触发不必要的滚动更新。

kubectl rollout pause  deployment/deployment-name

暂停 Deployment

kubectl rollout resumedeployment/deployment-name  

继续（resume）该 Deployment，可使前面所有的变更一次性生效



#####查看



