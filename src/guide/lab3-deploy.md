# 部署应用及发布服务

## 在EKS集群上发布Load Balancer服务

### 部署Nginx应用

```bash
cat <<EoF > ~/environment/run-my-nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
  namespace: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EoF

# 创建命名空间
kubectl create ns my-nginx
# 部署应用
kubectl -n my-nginx apply -f ~/environment/run-my-nginx.yaml
kubectl -n my-nginx get pods -o wide
```

查看Pod IP

```bash
kubectl -n my-nginx get pods -o yaml | grep 'podIP:'
```

发布服务

```bash
kubectl -n my-nginx expose deployment/my-nginx
```

查看服务类型（ClusterIP）及终端节点（Endpoints）

```bash
kubectl -n my-nginx describe svc my-nginx
```

将服务类型从默认的ClusterIP修改为LoadBalancer

```bash
kubectl -n my-nginx patch svc my-nginx -p '{"spec": {"type": "LoadBalancer"}}'
kubectl -n my-nginx get svc my-nginx
```

查看服务类型

```bash
kubectl -n my-nginx get svc my-nginx
```

上述命令将my-nginx服务发布在AWS Classic Load Balancer（CLB）上，CLB的创建需要等待30秒钟左右，可以在控制台的EC2服务的负载均衡器页面查看刚刚创建的CLB。创建完毕后可以通过CLB外部域名访问服务

```bash
export loadbalancer=$(kubectl -n my-nginx get svc my-nginx -o jsonpath='{.status.loadBalancer.ingress[*].hostname}')

curl -k -s http://${loadbalancer} | grep title
```

返回Welcome to nginx!表示实验成功。

## 在Application Load Balancer上发布Ingress

#### 创建 IAM OIDC provider

```bash
eksctl utils associate-iam-oidc-provider \
    --cluster=$CLUSTER_NAME \
    --approve
```

#### 

### 安装 AWS Load Balancer Controller

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
    --policy-document file://iam_policy.json
```

记录创建出来的 policy 的 arn, 在接下来的命令可以用上。

### 创建 IAM Role 和 ServiceAccount 

更新如下命令参数指定的 policy arn，并执行该命令创建 iam role 和 service account

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query "Account" --output text)

eksctl create iamserviceaccount \
       --cluster=$CLUSTER_NAME \
       --namespace=kube-system \
       --name=aws-load-balancer-controller \
       --attach-policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/AWSLoadBalancerControllerIAMPolicy \
       --override-existing-serviceaccounts \
       --approve
```


### 安装 aws-load-balancer-controller

使用Helm工具作为安装aws-load-balancer-controller的工具

```bash
curl -sSL https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
helm version --short
```

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



### 部署示例应用及发布Ingress

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

从 ADDRESS 这一列可以看到 ALB 的地址，等待30秒左右ALB创建成功后，在浏览器中访问该地址即可查看部署的示例网站

### 参考文档

- https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/deploy/installation/
- https://www.eksworkshop.com/beginner/130_exposing-service/ingress_controller_alb/
- https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html