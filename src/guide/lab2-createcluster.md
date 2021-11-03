# 创建EKS集群

## 创建集群描述文件



```bash
cat << EOF > eksworkshop.yaml
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: ${ClUSTER_NAME}
  region: ${AWS_REGION}
  version: "1.19"

availabilityZones: ["${AZS[0]}", "${AZS[1]}", "${AZS[2]}"]

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


## 查看EKS集群工作节点

  ```bash
   kubectl cluster-info
   kubectl get node
   ```
  