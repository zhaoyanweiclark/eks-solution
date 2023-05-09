## How to deploy EKS app from private ECR

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

Here is a demo depoying from private ECR with repository-name=`api-helloworld` and cluster-name=`stqd-demo`

1. Create Fargate profile
```bash
$ eksctl create fargateprofile --cluster stqd-demo --region ap-southeast-2 --name helloworld --namespace helloworld
```

2. Create ECR private repos
```bash
$ aws ecr create-repository --repository-name api-helloworld --region ap-southeast-2
```

3. Build image with a local Dockerfile -> `./Dockerfile`
```bash
$ aws sts get-caller-identity --query Account | docker build -t $(xargs).dkr.ecr.ap-southeast-2.amazonaws.com/api-helloworld:latest .
```

4. Login ECR and push image
```bash
$ aws ecr get-login-password --region ap-southeast-2 | \
  docker login  --username AWS --password-stdin $(aws sts get-caller-identity --query Account | xargs).dkr.ecr.ap-southeast-2.amazonaws.com
$ aws sts get-caller-identity --query Account | docker push $(xargs).dkr.ecr.ap-southeast-2.amazonaws.com/api-helloworld:latest
```

5. Deploymetn with kubectl command
```bash
$ kubectl apply -f helloworld.yaml
```

6. After a while, make sure game deployed
```bash
$ kubectl get deployments -n helloworld
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
deployment-2048   3/3     3            3           108s
$ kubectl get ing -n helloworld
NAME           CLASS   HOSTS   ADDRESS                                                             PORTS   AGE
ingress-2048   alb     *       k8s-hellowor-ingressh-xxxx-yyyy.ap-southeast-2.elb.amazonaws.com/   80      2m39s
```

7. Open browser and input `http://k8s-hellowor-ingressh-xxxx-yyyy.ap-southeast-2.elb.amazonaws.com/`
