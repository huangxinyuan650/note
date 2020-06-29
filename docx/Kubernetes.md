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
#### ETCD升级（3.3->3.4）
- step1:下载etcd3.4.3（https://github.com/etcd-io/etcd/releases/download/v3.4.3/etcd-v3.4.3-linux-amd64.tar.gz），解压 tar -zxvf etcd-v3.4.3-linux-amd64.tar.gz
- step2: 停掉kube-apiserver，systemctl stop kube-apiserver
- step3: etcd原数据备份，etcdctl backup --data-dir=/var/lib/etcd --backup-dir=/root/etcd_backup/back_data
- step4: 停掉etcd服务，systemctl stop etcd
- step5: 更新etcd二进制文件，cd etcd-v3.4.3-linux-amd64 && cp etcd etcdctl /opt/kube/bin
- step6: 修改etcd service配置参数，添加--initial-cluster-state new \ --logger zap \ --log-outputs stderr \ --heartbeat-interval=100 \ --election-timeout=500 \ --snapshot-count=5000 \
- step7: 恢复etcd备份数据，etcd -data-dir=/root/etcd_backup/back_data --force-new-cluster 待执行完成时，终止恢复程序
- step8: 启动etcd服务，systemctl daemon-reload && systemctl start etcd && systemctl start kube-apiserver
- step9: 确认服务正常，kubectl get pod