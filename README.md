## How to use Kubernetes

### References

- <https://github.com/zhaoyanweiclark/kubernetes-course>

### Kubernetes

#### kubernetes概念

- Cluster
- Master：集群控制节点，每个集群需要至少一个master节点负责集群的管控
- Worker：工作负载节点，由master分配容器到这些node工作节点上，然后node节点上的docker负责容器的运行
- Pod：kubernetes的最小控制单元，容器都是运行在pod中的，一个pod中可以有1个或者多个容器
- Controller：控制器，通过它来实现对pod的管理，比如启动pod、停止pod、伸缩pod的数量等等
- Service：pod对外服务的统一入口，下面可以维护者同一类的多个pod
- Label：标签，用于对pod进行分类，同一类pod会拥有相同的标签
- NameSpace：命名空间，用来隔离pod的运行环境

#### kubernetes集群环境搭建

多台 Linux 服务器，在生产环境一般是多主多从

#### 集群管理方式

- 命令式对象管理：直接使用命令去操作kubernetes资源
```bash
kubectl run nginx-pod --image=nginx:1.17.1 --port=80
```

- 命令式对象配置：通过命令配置和配置文件去操作kubernetes资源
```bash
kubectl create/patch -f nginx-pod.yaml
```

- 声明式对象配置：通过apply命令和配置文件去操作kubernetes资源
```bash
kubectl apply -f nginx-pod.yaml
```

- `kubectl` 是客户端命令

#### 控制器

- ReplicationController：比较原始的pod控制器，已经被废弃，由ReplicaSet替代
- ReplicaSet：保证副本数量一直维持在期望值，并支持pod数量扩缩容，镜像版本升级
- Deployment：通过控制ReplicaSet来控制Pod，并支持滚动升级、回退版本
- Horizontal Pod Autoscaler：可以根据集群负载自动水平调整Pod的数量，实现削峰填谷
- DaemonSet：在集群中的指定Node上运行且仅运行一个副本，一般用于守护进程类的任务
- Job：它创建出来的pod只要完成任务就立即退出，不需要重启或重建，用于执行一次性任务
- Cronjob：它创建的Pod负责周期性任务控制，不需要持续后台运行
- StatefulSet：管理有状态应用

#### Pod

- 每个Pod中都可以包含一个或者多个容器
- 查看 Pod 资源对象的属性 `kubectl explain pods`

#### Service

TCP/IP 4层网络代理
在 EKS 中 service 可以连接 NLB

- ClusterIP：默认值，它是Kubernetes系统自动分配的虚拟IP，只能在集群内部访问
- NodePort：将Service通过指定的Node上的端口暴露给外部，通过此方法，就可以在集群外部访问服务
- LoadBalancer：使用外接负载均衡器完成到服务的负载分发，注意此模式需要外部云环境支持
- ExternalName： 把集群外部的服务引入集群内部，直接使用

#### Ingress

HTTP/HTTPS 7层网络代理
在 EKS 中 ingress 可以连接 ALB

### How to create aws eks infrastructures

- `initialize/`

### How to deploy EKS app from public ECR

- `game-2048/`

### How to deploy EKS app from private ECR

- `helloworld/`
