## How to create AWS EKS Fargate infrastructure

### References

- <https://docs.aws.amazon.com/zh_cn/eks/latest/userguide/alb-ingress.html>
- <https://docs.aws.amazon.com/zh_cn/eks/latest/userguide/aws-load-balancer-controller.html>

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

4. Install helm
Form <https://helm.sh/docs/intro/install/>
```bash
$ helm version
version.BuildInfo{Version:"v3.11.3", GitCommit:"xxxxxx", GitTreeState:"clean", GoVersion:"go1.20.3"}
```

5. Config aws cli
```bash
$ aws configure
AWS Access Key ID [****************HH64]: 
AWS Secret Access Key [****************tmA+]: 
Default region name [ap-southeast-2]: 
Default output format [None]:
```

### Launch Steps

Here is a demo to create a EKS with cluster-name=`stqd-demo`, region=`ap-southeast-2`

1. Create EKS cluster
```bash
$ eksctl create cluster --name stqd-demo --region ap-southeast-2 --fargate
```

2. Enable cluster OIDC
```bash
$ eksctl utils associate-iam-oidc-provider --region=ap-southeast-2 --cluster=stqd-demo --approve
```

3. Install Addons (If you cluster version > 1.21)
```bash
$ eksctl create addon --cluster stqd-demo --name coredns --version latest --force
$ eksctl create addon --cluster stqd-demo --name kube-proxy --version latest --force
$ eksctl create addon --cluster stqd-demo --name vpc-cni --version latest --force
```

4. Create ALB IAM policy if not exists
```bash
$ aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://./iam_policy.json
```

5. Attach the polic to cluster
```bash
$ aws sts get-caller-identity --query Account\
  | eksctl create iamserviceaccount \
  --cluster=stqd-demo \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::$(xargs):policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

6. Add EKS repos
```bash
$ helm repo add eks https://aws.github.io/eks-charts
$ helm repo update
```

7. Install ALB controller for EKS
```bash
$ aws eks describe-cluster --name stqd-demo --query cluster.resourcesVpcConfig.vpcId \
 | helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=stqd-demo \
  --set region=ap-southeast-2 \
  --set vpcId=$(xargs) \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

8. Make sure account added & ALB controller installed
```bash
$ kubectl config get-contexts | grep stqd-demo.ap-southeast-2.eksctl.io
...   stqd-demo.ap-southeast-2.eksctl.io   ...

$ kubectl get deployment -n kube-system aws-load-balancer-controller
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
aws-load-balancer-controller   2/2     2            2           101s
```

9. Done!

### Deletion

```bash
$ eksctl delete cluster --name stqd-demo
```
