# 创建EKS集群

打开Cloud9终端管理控制台， 使用eksctl 创建EKS集群(操作需要10-15分钟),该命令同时会创建名字为eksworkshop,版本为v1.20的EKS 集群，同时创建一个包含2个m5.large 实例的受管节点组。

 ```bash
 export CLUSTER_NAME=eksworkshop
 echo "export CLUSTER_NAME=${CLUSTER_NAME}" >> ~/.bashrc
 eksctl create cluster \
       --name $CLUSTER_NAME \
       --version 1.20 \
       --managed
 ```

 ![](media/15764759782724/15764761011094.jpg)

  查看EKS集群工作节点
  ```bash
   kubectl cluster-info
   kubectl get node
  ```
  ![](media/15764759782724/15764762619982.jpg)
