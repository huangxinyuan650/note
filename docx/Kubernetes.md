## k8s

**基础知识**
* Pod：服务单元
* RC：复制控制器Replication Controller 控制Pod的数量
* RS：副本集Replica Set 升级版RC（支持更多匹配模式）
* Deployment：部署，新建，更新
* Service：服务（Kube-proxy是K8s集群内部的负载均衡器），一个Service对应一个集群内部有效虚拟IP，内部通过Kube-proxy来完成内部虚拟IP访问服务
* ETCD:一个键值存储仓库,用于配置共享和服务发现

**K8S install**
* docker
* etcd
* kubeadm 
* kubelet 
* kubectl
* flannel网络
* ingress

**K8S服务重启(组件)**
```shell
systemctl restart/enable/status kube-apiserver.service
systemctl restart/enable/status kube-controller-manager.service
systemctl restart/enable/status kube-scheduler.service
```
ectd谨慎重启

**启动POD**
将制定的pod按配置文件启动，一般如果pod已经起来会根据配置文件的变动重新启动
```Shell
kubectl apply -f 28.app.yaml
```
根据指定的配置文件删除pod
```Shell
kubectl delete -f 28.app.yaml
```