# 部署应用

## 安装 AWS Load Balancer Controller

### 准备权限

#### 创建 IAM OIDC provider

```bash
eksctl utils associate-iam-oidc-provider \
    --cluster=$CLUSTER_NAME \
    --approve
```

#### 创建 IAM Policy


```bash
curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.3.0/docs/install/iam_policy.json
```

```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```

记录创建出来的 policy 的 arn, 在接下来的命令可以用上。

### 创建 IAM Role 和 ServiceAccount 

更新如下命令参数指定的 policy arn，并执行该命令创建 iam role 和 service account
```bash
eksctl create iamserviceaccount \
       --cluster=$CLUSTER_NAME \
       --namespace=kube-system \
       --name=aws-load-balancer-controller \
       --attach-policy-arn=arn:aws:iam::59********44:policy/AWSLoadBalancerControllerIAMPolicy \
       --override-existing-serviceaccounts \
       --approve
```


### 安装 aws-load-balancer-controller

接下来使用 Helm 来进行安装：

#### 添加 eks-charts 并更新 Helm 仓库

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```
#### 安装

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=${CLUSTER_NAME} \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller 
```

#### 检查安装结果

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```

输出：
```bash
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
aws-load-balancer-controller   2/2     2            2           84s
```



### 部署示例应用

#### 部署

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.3.0/docs/examples/2048/2048_full.yaml
```

### 检查 ingress
```bash
kubectl get ingress/ingress-2048 -n game-2048
```

输出：
```bash
NAME           CLASS    HOSTS   ADDRESS                                                                   PORTS   AGE
ingress-2048   <none>   *       k8s-game2048-ingress2-xxxxxxxxxx-yyyyyyyyyy.us-west-2.elb.amazonaws.com   80      2m32s
```

从 ADDRESS 这一列可以看到 ALB 的地址，在浏览器中访问该地址即可查看部署的示例网站

### 参考文档

- https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/deploy/installation/
- https://www.eksworkshop.com/beginner/130_exposing-service/ingress_controller_alb/
- https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html