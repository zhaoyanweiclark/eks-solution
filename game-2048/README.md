## How to deploy EKS app from public ECR

### Requirements

1. Install aws cli
From <https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html>
```bash
$ aws --version
aws-cli/2.11.11 Python/3.11.2 Darwin/22.3.0 exe/x86_64 prompt/off
```

2. Install aws eksctl
From <https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html>
```bash
$ eksctl version
0.137.0
```

3. Install kubectl
Install docker desktop from <https://docs.docker.com/desktop/>, then enable kubernetes
```bash
$ kubectl version
Kustomize Version: v4.5.7
Server Version: xxxxxxx ...
```

4. Config aws cli
```bash
$ aws configure
AWS Access Key ID [****************HH64]: 
AWS Secret Access Key [****************tmA+]: 
Default region name [ap-southeast-2]: 
Default output format [None]:
```

5. Complete EKS infrastructure deployment with cluster-name=`stqd-demo`
From <>

### Launch Steps

Here is a demo deploying game-2048 from public ECR with cluster-name=`stqd-demo`

1. Create Fargate profile
```bash
$ eksctl create fargateprofile --cluster stqd-demo --region ap-southeast-2 --name game-2048 --namespace game-2048
```

2. Deploymetn with kubectl command
```bash
$ kubectl apply -f 2048.yaml
```

3. After a while, make sure game deployed
```bash
$ kubectl get deployments -n game-2048
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
deployment-2048   3/3     3            3           108s

$ kubectl get ing -n game-2048
NAME           CLASS   HOSTS   ADDRESS                                                            PORTS   AGE
ingress-2048   alb     *       k8s-game2048-ingress2-xxxx.yyyy.ap-southeast-2.elb.amazonaws.com   80      2m39s
```

4. Open browser and input `http://k8s-game2048-ingress2-xxxx.yyyy.ap-southeast-2.elb.amazonaws.com/`
