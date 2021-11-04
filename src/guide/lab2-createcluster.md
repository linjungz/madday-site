# 创建EKS集群

## 创建集群描述文件



```bash
cat << EOF > eksworkshop.yaml
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: ${CLUSTER_NAME}
  region: ${AWS_REGION}
  version: "1.19"

managedNodeGroups:
- name: nodegroup
  desiredCapacity: 3
  instanceType: t3.small

EOF
```

这个描述文件会创建一个v1.19版本的 EKS 集群，同时创建一个包含 3个t3.small 实例的托管节点组。

## 创建集群

 ```bash
 eksctl create cluster -f eksworkshop.yaml
 ```


## 通过命令行查看EKS集群工作节点

  ```bash
   kubectl cluster-info
   kubectl get node
  ```

## 在管理控制台上查看EKS集群

将当前登录管理控制台用户加入EKS集群的管理员组，方便在控制台上查看EKS集群的详细信息

  ```bash
# 如果实验中Cloud9环境不是在us-west-2开启下列命令会出现NotFoundException的错误，表示在us-west-2的区域中无法查到改Cloud9环境。可以通过手动设置rolearn环境变量为当前登录管理控制台用户ARN的方法解决
c9builder=$(aws cloud9 describe-environment-memberships --environment-id=$C9_PID | jq -r '.memberships[].userArn')
if echo ${c9builder} | grep -q user; then
	rolearn=${c9builder}
        echo Role ARN: ${rolearn}
elif echo ${c9builder} | grep -q assumed-role; then
        assumedrolename=$(echo ${c9builder} | awk -F/ '{print $(NF-1)}')
        rolearn=$(aws iam get-role --role-name ${assumedrolename} --query Role.Arn --output text) 
        echo Role ARN: ${rolearn}
fi

eksctl create iamidentitymapping --cluster ${CLUSTER_NAME} --arn ${rolearn} --group system:masters --username admin
  ```

在控制台上查看EKS集群节点、网络、工作负载等信息。
