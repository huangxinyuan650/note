#### Master节点安装
- step1: 设置机器免密登录
```shell
ssh-keygen  ### 一直回车到最后
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys ###公钥加入到authorized_keys中
systemctl restart sshd  ### 重启sshd服务
ssh localhost   ### 测试是否免密登录自己
```

- step2: 关闭防火墙
```shell
systemctl stop firewalld && systemctl disable firewalld
```
- step3: 关闭系统交换分区和SeLinux
```shell
swapoff -a
free -h ### 确认交换分区是否关闭
# SeLinux一般都是关闭状态，若开启自行百度关闭方式
```
- step4: 安装NTP
```shell
# ubuntu
apt-get install ntp -y
# centos
yum install ntp -y
# 设置开启自启动
systemctl enable ntpd && systemctl start ntpd
```
- step5: 安装配置CFSSL（构建本地CA证书）
```shell
# 创建存放目录
mkdir -p /home/work/_src
cd /home/work/_src
# 下载软件包
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
# 授权变成二进制文件
chmod +x cfssl_linux-amd64 cfssljson_linux-amd64 cfssl-certinfo_linux-amd64
# 拷贝软件至相应目录
mv cfssl_linux-amd64 /usr/local/bin/cfssl
mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
mv cfssl-certinfo_linux-amd64 /usr/bin/cfssl-certinfo
```
- step6：创建etcd和kubernetes的安装目录并确认系统内核
```shell
# etcd和kubernetes安装目录创建
mkdir /home/work/_app/k8s/etcd/{bin,cfg,ssl} -p
mkdir /home/work/_app/k8s/kubernetes/{bin,cfg,ssl,ssl_cert} -p
mkdir /home/work/_data/etcd -p
# 系统内核确认
hostnamectl ### 确认kernel版本高于3.19即可，否则升级内核
```
- step7: 安装docker（16.04和18.04稍微有点差异）
```shell
######### Ubuntu18.04安装方式
# step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
# step 2: 安装GPG证书
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# Step 3: 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# Step 4: 更新并安装 Docker-CE
sudo apt-get -y update
sudo apt-get -y install docker-ce
########## Ubuntu16.04安装方式
# 1. 清除之前安装的
sudo apt-get remove docker docker-engine docker.io
# 2. 更新系统库
sudo apt-get update
# 3. 安装curl等工具
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
# 4. 添加镜像源key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
# 5. 添加镜像源
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
# 6. 安装docker-ce
sudo apt-get install docker-ce

####设置docker开机启动
systemctl enable docker && systemctl start docker
```
- step8: 安装etcd3.10
```shell
### 1、生成 ETCD SERVER 证书用到的JSON请求文件
mkdir -p /home/work/_src/ssl_etcd
cd /home/work/_src/ssl_etcd

cat << EOF | tee ca-config.json
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "etcd": {
         "expiry": "87600h",
         "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ]
      }
    }
  }
}
EOF
###文件参数说明
### 默认策略，指定了证书的有效期是10年(87600h)
### etcd策略，指定了证书的用途
### signing, 表示该证书可用于签名其它证书；生成的 ca.pem 证书中 CA=TRUE
### server auth：表示 client 可以用该 CA 对 server 提供的证书进行验证
### client auth：表示 server 可以用该 CA 对 client 提供的证书进行验证

### 2、创建 ETCD CA 证书配置文件
cat << EOF | tee ca-csr.json
{
    "CN": "etcd CA",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Beijing",
            "ST": "Beijing"
        }
    ]
}
EOF

### 3、创建 ETCD SERVER 证书配置文件
cat << EOF | tee server-csr.json
{
    "CN": "etcd",
    "hosts": [
    "10.0.0.100"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Beijing",
            "ST": "Beijing"
        }
    ]
}
EOF
### 配置文件说明
### hosts中填写master节点和node节点（若单节点只需填写master节点ip即可）

### 4、生成 ETCD CA 证书和私钥
cfssl gencert -initca ca-csr.json | cfssljson -bare ca

### 5、生成 ETCD SERVER 证书和私钥
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=etcd server-csr.json | cfssljson -bare server
# 拷贝证书到指定目录
cp server.pem server-key.pem /home/work/_app/k8s/etcd/ssl/
cp *.pem /home/work/_app/k8s/etcd/ssl/

### 6、安装 ETCD
# 下载etcd安装包、解压、拷贝etcd、etcdctl到指定目录
cd /home/work/_src/
wget https://github.com/etcd-io/etcd/releases/download/v3.3.10/etcd-v3.3.10-linux-amd64.tar.gz
tar -xvf etcd-v3.3.10-linux-amd64.tar.gz
cd etcd-v3.3.10-linux-amd64
cp etcd etcdctl /home/work/_app/k8s/etcd/bin/

### 7、创建etcd系统启动文件
vim /lib/systemd/system/etcd.service 
# centos则为/usr/lib/systemd/system/etcd.service
### etcd.service文件内容开始
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
EnvironmentFile=/home/work/_app/k8s/etcd/cfg/etcd.conf
ExecStart=/home/work/_app/k8s/etcd/bin/etcd \
--name=${ETCD_NAME} \
--data-dir=${ETCD_DATA_DIR} \
--listen-peer-urls=${ETCD_LISTEN_PEER_URLS} \
--listen-client-urls=${ETCD_LISTEN_CLIENT_URLS},http://127.0.0.1:2379 \
--advertise-client-urls=${ETCD_ADVERTISE_CLIENT_URLS} \
--initial-advertise-peer-urls=${ETCD_INITIAL_ADVERTISE_PEER_URLS} \
--initial-cluster=${ETCD_INITIAL_CLUSTER} \
--initial-cluster-token=${ETCD_INITIAL_CLUSTER_TOKEN} \
--initial-cluster-state=new \
--cert-file=/home/work/_app/k8s/etcd/ssl/server.pem \
--key-file=/home/work/_app/k8s/etcd/ssl/server-key.pem \
--peer-cert-file=/home/work/_app/k8s/etcd/ssl/server.pem \
--peer-key-file=/home/work/_app/k8s/etcd/ssl/server-key.pem \
--trusted-ca-file=/home/work/_app/k8s/etcd/ssl/ca.pem \
--peer-trusted-ca-file=/home/work/_app/k8s/etcd/ssl/ca.pem
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
### etcd.service文件内容结束

### 8、创建etcd主配置文件
cat << EOF | tee /home/work/_app/k8s/etcd/cfg/etcd.conf
#[Member]
# ETCD的节点名
ETCD_NAME="etcd00"
# ETCD的数据存储目录
ETCD_DATA_DIR="/home/work/_data/etcd"
# 该节点与其他节点通信时所监听的地址列表，多个地址使用逗号隔开，其格式可以划分为scheme://IP:PORT，这里的scheme可以是http、https
ETCD_LISTEN_PEER_URLS="https://10.0.0.100:2380"
# 该节点与客户端通信时监听的地址列表
ETCD_LISTEN_CLIENT_URLS="https://10.0.0.100:2379"
 
#[Clustering]
# 该成员节点在整个集群中的通信地址列表，这个地址用来传输集群数据的地址。因此这个地址必须是可以连接集群中所有的成员的。
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://10.0.0.100:2380"
# 配置集群内部所有成员地址，其格式为：ETCD_NAME=ETCD_INITIAL_ADVERTISE_PEER_URLS，如果有多个使用逗号隔开
ETCD_ADVERTISE_CLIENT_URLS="https://10.0.0.100:2379"
# cluster节点多个实用逗号隔开（单节点只需master节点IP即可）
ETCD_INITIAL_CLUSTER="etcd00=https://10.0.0.100:2380"
# 初始化集群token
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
# 初始化集群状态，new表示新建
ETCD_INITIAL_CLUSTER_STATE="new"

#[Security]
ETCD_CERT_FILE="/home/work/_app/k8s/etcd/ssl/server.pem"
ETCD_KEY_FILE="/home/work/_app/k8s/etcd/ssl/server-key.pem"
ETCD_TRUSTED_CA_FILE="/home/work/_app/k8s/etcd/ssl/ca.pem"
ETCD_CLIENT_CERT_AUTH="true"
ETCD_PEER_CERT_FILE="/home/work/_app/k8s/etcd/ssl/server.pem"
ETCD_PEER_KEY_FILE="/home/work/_app/k8s/etcd/ssl/server-key.pem"
ETCD_PEER_TRUSTED_CA_FILE="/home/work/_app/k8s/etcd/ssl/ca.pem"
ETCD_PEER_CLIENT_CERT_AUTH="true"

EOF

### 9、启动etcd服务、检查etcd运行状态、查看etcd集群成员
# 启动etcd
systemctl daemon-reload && systemctl enable etcd && systemctl start etcd
# 检查etcd运行状态
/home/work/_app/k8s/etcd/bin/etcdctl --ca-file=/home/work/_app/k8s/etcd/ssl/ca.pem --cert-file=/home/work/_app/k8s/etcd/ssl/server.pem --key-file=/home/work/_app/k8s/etcd/ssl/server-key.pem cluster-health
# 查看etcd集群成员
/home/work/_app/k8s/etcd/bin/etcdctl --ca-file=/home/work/_app/k8s/etcd/ssl/ca.pem --cert-file=/home/work/_app/k8s/etcd/ssl/server.pem --key-file=/home/work/_app/k8s/etcd/ssl/server-key.pem  member list

```

- step9、安装flannl v1.11.0
```shell
### 1、向 ETCD 集群写入网段信息
/home/work/_app/k8s/etcd/bin/etcdctl --ca-file=/home/work/_app/k8s/etcd/ssl/ca.pem --cert-file=/home/work/_app/k8s/etcd/ssl/server.pem --key-file=/home/work/_app/k8s/etcd/ssl/server-key.pem --endpoints="https://10.0.0.100:2379"  set /coreos.com/network/config  '{ "Network": "10.244.0.0/16", "Backend": {"Type": "vxlan"}}'
### 参数说明：若为多节点则endpoints添加多个地址使用逗号隔开，单节点只需master节点即可；写入的 Pod 网段 ${CLUSTER_CIDR} 必须是 /16 段地址，必须与 kube-controller-manager 的 –cluster-cidr 参数值一致；

### 2、安装 Flannel
cd /home/work/_src
wget https://github.com/coreos/flannel/releases/download/v0.11.0/flannel-v0.11.0-linux-amd64.tar.gz
tar -xvf flannel-v0.11.0-linux-amd64.tar.gz
mv flanneld mk-docker-opts.sh /home/work/_app/k8s/kubernetes/bin/

### 3、配置flannel
# 创建 /home/work/_app/k8s/kubernetes/cfg/flanneld
#### flannel文件内容开始

FLANNEL_OPTIONS="--etcd-endpoints=https://10.0.0.100:2379 -etcd-cafile=/home/work/_app/k8s/etcd/ssl/ca.pem -etcd-certfile=/home/work/_app/k8s/etcd/ssl/server.pem -etcd-keyfile=/home/work/_app/k8s/etcd/ssl/server-key.pem"

#### flannel文件内容结束
##参数说明：若多节点则endpoints中添加节点信息用逗号隔开，单节点只需添加master即可

### 4、创建 Flannel 系统启动文件
#### flanneld.service文件内容开始

[Unit]
Description=Flanneld overlay address etcd agent
After=network-online.target network.target
Before=docker.service

[Service]
Type=notify
EnvironmentFile=/home/work/_app/k8s/kubernetes/cfg/flanneld
ExecStart=/home/work/_app/k8s/kubernetes/bin/flanneld --ip-masq $FLANNEL_OPTIONS
ExecStartPost=/home/work/_app/k8s/kubernetes/bin/mk-docker-opts.sh -k DOCKER_NETWORK_OPTIONS -d /run/flannel/subnet.env
Restart=on-failure

[Install]
WantedBy=multi-user.target

#### flanneld.service文件内容结束
####参数说明：mk-docker-opts.sh 脚本将分配给 Flanneld 的 Pod 子网网段信息写入 /run/flannel/docker 文件，后续 Docker 启动时 使用这个文件中的环境变量配置 docker0 网桥.
Flanneld 使用系统缺省路由所在的接口与其它节点通信，对于有多个网络接口（如内网和公网）的节点，可以用 -iface 参数指定通信接口;

### 5、配置 Docker 启动指定子网段
#### 修改/lib/systemd/system/docker.service（centos文件路径为/usr/lib/systemd/system/docker.service）
#### 在service中的ExecStart前添加 EnvironmentFile=/run/flannel/subnet.env，并且在ExecStart的值后面添加$DOCKER_NETWORK_OPTIONS作为参数

### 6、启动服务、查看Flannel 服务设置 docker0 网桥状态、验证 Flannel 服务
###启动服务
systemctl daemon-reload && systemctl stop docker && systemctl enable flanneld && systemctl start flanneld && systemctl start docker
###查看Flannel 服务设置 docker0 网桥状态
ip add
###验证 Flannel 服务
cat /run/flannel/subnet.env
```

- setp10：Kubernetes安装
```shell
### 1、生成 Kubernetes 证书请求的JSON请求文件
cd /home/work/_app/k8s/kubernetes/ssl/
cat << EOF | tee ca-config.json
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "server": {
        "usages": [
          "signing",
          "key encipherment",
          "server auth"
        ],
        "expiry": "8760h"
      },
      "client": {
        "usages": [
          "signing",
          "key encipherment",
          "client auth"
        ],
        "expiry": "8760h"
      }
    }
  }
}
EOF

### 2、生成 Kubernetes CA 配置文件和证书
cat << EOF | tee ca-csr.json
{
    "CN": "kubernetes CA",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Beijing",
            "ST": "Beijing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF
### 初始化一个 Kubernetes CA 证书
cfssl gencert -initca ca-csr.json | cfssljson -bare ca

### 3、生成 Kube API Server 配置文件和证书
cat << EOF | tee kube-apiserver-server-csr.json
{
    "CN": "kubernetes",
    "hosts": [
      "127.0.0.1",
      "10.0.0.1",
      "10.0.0.100",
      "kubernetes",
      "kubernetes.default",
      "kubernetes.default.svc",
      "kubernetes.default.svc.cluster",
      "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Beijing",
            "ST": "Beijing",
            "O": "k8s",
            "OU": "API Server"
        }
    ]
}
EOF
#### 参数说明：hosts中10.0.0.1为网关地址，10.0.0.100为master地址
### 生成 kube-apiserver 证书
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=server kube-apiserver-server-csr.json | cfssljson -bare kube-apiserver-server

### 4、生成 kubelet client 配置文件和证书
cat << EOF | tee kubelet-client-csr.json
{
  "CN": "kubelet",
  "hosts": [""],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "L": "Beijing",
      "O": "k8s",
      "OU": "Kubelet",
      "ST": "Beijing"
    }
  ]
}
EOF

### 生成 kubelet client证书
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=client kubelet-client-csr.json | cfssljson -bare kubelet-client

### 5、生成 Kube-Proxy 配置文件和证书
cat << EOF | tee kube-proxy-client-csr.json
{
  "CN": "system:kube-proxy",
  "hosts": [""],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "L": "Beijing",
      "O": "k8s",
      "OU": "System",
      "ST": "Beijing"
    }
  ]
}
EOF
### 生成 Kube-Proxy 证书
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=client kube-proxy-client-csr.json | cfssljson -bare kube-proxy-client

### 6、生成 kubectl 管理员配置文件和证书
#### 创建 kubectl 管理员证书配置文件
cat << EOF | tee kubernetes-admin-user.csr.json
{
  "CN": "admin",
  "hosts": [""],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "L": "Beijing",
      "O": "k8s",
      "OU": "Cluster Admins",
      "ST": "Beijing"
    }
  ]
}
EOF

#### 生成 kubectl 管理员证书
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=client kubernetes-admin-user.csr.json | cfssljson -bare kubernetes-admin-user

```

- step11： 部署 Kubernetes Master 节点并加入集群
```shell

### 1、下载文件并安装 Kubernetes Server
cd /home/work/_src/
####可能出现无法下载情况（国内），需通过其他途径下载拷贝到响应目录下（kubectl、kubelet可以直接在国内镜像站点下载到，kube-scheduler kube-apiserver kube-controller-manager kube-proxy可以通过docker镜像的方式启动也可以通过境外服务器下载好整个包，此处描述的为下载好整个包的方式）
wget https://dl.k8s.io/v1.13.0/kubernetes-server-linux-amd64.tar.gz
tar -xzvf kubernetes-server-linux-amd64.tar.gz
cd kubernetes/server/bin/
cp kube-scheduler kube-apiserver kube-controller-manager kubectl kubelet kube-proxy /home/work/_app/k8s/kubernetes/bin/

### 2、部署 Apiserver
#### 创建 TLS Bootstrapping Token（记录下来 token_value）
head -c 16 /dev/urandom | od -An -t x | tr -d ' '

#### 创建 /home/work/_app/k8s/kubernetes/cfg/token-auth-file
vim /home/work/_app/k8s/kubernetes/cfg/token-auth-file
#### token-auth-file文件内容开始
token_value,kubelet-bootstrap,10001,"system:kubelet-bootstrap"
#### token-auth-file文件内容结束

#### 创建 Apiserver 配置文件
vim /home/work/_app/k8s/kubernetes/cfg/kube-apiserver
##### kube-apiserver文件内容开始
KUBE_APISERVER_OPTS="--logtostderr=true \
--v=4 \
--etcd-servers=https://10.0.0.100:2379 \
--bind-address=10.0.0.100 \
--secure-port=6443 \
--advertise-address=10.0.0.100 \
--allow-privileged=true \
--service-cluster-ip-range=10.244.0.0/16 \
--enable-admission-plugins=NamespaceLifecycle,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota,NodeRestriction \
--authorization-mode=RBAC,Node \
--enable-bootstrap-token-auth \
--token-auth-file=/home/work/_app/k8s/kubernetes/cfg/token-auth-file \
--service-node-port-range=30000-50000 \
--tls-cert-file=/home/work/_app/k8s/kubernetes/ssl/kube-apiserver-server.pem  \
--tls-private-key-file=/home/work/_app/k8s/kubernetes/ssl/kube-apiserver-server-key.pem \
--client-ca-file=/home/work/_app/k8s/kubernetes/ssl/ca.pem \
--service-account-key-file=/home/work/_app/k8s/kubernetes/ssl/ca-key.pem \
--etcd-cafile=/home/work/_app/k8s/etcd/ssl/ca.pem \
--etcd-certfile=/home/work/_app/k8s/etcd/ssl/server.pem \
--etcd-keyfile=/home/work/_app/k8s/etcd/ssl/server-key.pem"
##### kube-apiserver文件内容结束
#### 参数说明：若为多节点则etcd-servers设置为多个使用逗号隔开

#### 创建 Apiserver 启动文件
#centos下为/usr/lib/systemd/system/kube-apiserver.service
vim /lib/systemd/system/kube-apiserver.service 
##### kube-apiserver.service文件内容开始
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
EnvironmentFile=-/home/work/_app/k8s/kubernetes/cfg/kube-apiserver
ExecStart=/home/work/_app/k8s/kubernetes/bin/kube-apiserver $KUBE_APISERVER_OPTS
Restart=on-failure

[Install]
WantedBy=multi-user.target

##### kube-apiserver.service文件内容结束

#### 启动 Kube Apiserver 服务、检查 Apiserver 服务是否运行

#### 启动 Kube Apiserver 服务
systemctl daemon-reload && systemctl enable kube-apiserver && systemctl start kube-apiserver

#### 检查 Apiserver 服务是否运行
systemctl status kube-apiserver

### 3、部署 Scheduler

#### 创建 /home/work/_app/k8s/kubernetes/cfg/kube-scheduler
vim /home/work/_app/k8s/kubernetes/cfg/kube-scheduler
##### kube-scheduler文件内容开始
KUBE_SCHEDULER_OPTS="--logtostderr=true --v=4 --master=127.0.0.1:8080 --leader-elect"
##### kube-scheduler文件内容结束

#### 创建 Kube-scheduler 系统启动文件
#centos下为/usr/lib/systemd/system/kube-scheduler.service
vim /lib/systemd/system/kube-scheduler.service
##### kube-scheduler.service文件内容开始
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
EnvironmentFile=-/home/work/_app/k8s/kubernetes/cfg/kube-scheduler
ExecStart=/home/work/_app/k8s/kubernetes/bin/kube-scheduler $KUBE_SCHEDULER_OPTS
Restart=on-failure

[Install]
WantedBy=multi-user.target
##### kube-scheduler.service文件内容结束

#### 启动 Kube-scheduler 服务
systemctl daemon-reload && systemctl enable kube-scheduler && systemctl start kube-scheduler
#### 检查 Kube-scheduler 服务是否运行
systemctl status kube-scheduler

### 4、部署 Kube-Controller-Manager 组件
#### 创建 /home/work/_app/k8s/kubernetes/cfg/kube-controller-manager
vim /home/work/_app/k8s/kubernetes/cfg/kube-controller-manager
#### kube-controller-manager文件内容开始
KUBE_CONTROLLER_MANAGER_OPTS="--logtostderr=true \
--v=4 \
--master=127.0.0.1:8080 \
--leader-elect=true \
--address=127.0.0.1 \
--service-cluster-ip-range=10.244.0.0/16 \
--cluster-name=kubernetes \
--cluster-signing-cert-file=/home/work/_app/k8s/kubernetes/ssl/ca.pem \
--cluster-signing-key-file=/home/work/_app/k8s/kubernetes/ssl/ca-key.pem  \
--root-ca-file=/home/work/_app/k8s/kubernetes/ssl/ca.pem \
--service-account-private-key-file=/home/work/_app/k8s/kubernetes/ssl/ca-key.pem"
#### kube-controller-manager文件内容结束

#### 创建 kube-controller-manager 系统启动文件
# centos下为/usr/lib/systemd/system/kube-controller-manager.service
vim /lib/systemd/system/kube-controller-manager.service
##### kube-controller-manager.service文件内容开始
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
EnvironmentFile=-/home/work/_app/k8s/kubernetes/cfg/kube-controller-manager
ExecStart=/home/work/_app/k8s/kubernetes/bin/kube-controller-manager $KUBE_CONTROLLER_MANAGER_OPTS
Restart=on-failure

[Install]
WantedBy=multi-user.target
##### kube-controller-manager.service文件内容结束

#### 启动 kube-controller-manager 服务
systemctl daemon-reload && systemctl enable kube-controller-manager && systemctl start kube-controller-manager

#### 检查 kube-controller-manager 服务是否运行
systemctl status kube-controller-manager

### 5、验证 API Server 服务
#### 将 kubectl 加入到$PATH变量中
echo "PATH=/home/work/_app/k8s/kubernetes/bin:$PATH:$HOME/bin" >> /etc/profile
source /etc/profile
kubectl get cs,nodes ### 查看节点状态

```

- step12: 部署 Kubelet
```shell
### 1、创建 bootstrap.kubeconfig、kube-proxy.kubeconfig 配置文件
vim /home/work/_app/k8s/kubernetes/cfg/env.sh
##### env.sh文件内容开始

#!/bin/bash
#创建kubelet bootstrapping kubeconfig 
BOOTSTRAP_TOKEN=4470210dbf9d9c57f8543bce4683c3ce
KUBE_APISERVER="https://10.0.0.100:6443"
#设置集群参数
kubectl config set-cluster kubernetes \
  --certificate-authority=/home/work/_app/k8s/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=bootstrap.kubeconfig
 
#设置客户端认证参数
kubectl config set-credentials kubelet-bootstrap \
  --token=${BOOTSTRAP_TOKEN} \
  --kubeconfig=bootstrap.kubeconfig
 
# 设置上下文参数
kubectl config set-context default \
  --cluster=kubernetes \
  --user=kubelet-bootstrap \
  --kubeconfig=bootstrap.kubeconfig
 
# 设置默认上下文
kubectl config use-context default --kubeconfig=bootstrap.kubeconfig
 
#----------------------

# 创建kube-proxy kubeconfig文件
 
kubectl config set-cluster kubernetes \
  --certificate-authority=/home/work/_app/k8s/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=kube-proxy.kubeconfig
 
kubectl config set-credentials kube-proxy \
  --client-certificate=/home/work/_app/k8s/kubernetes/ssl/kube-proxy-client.pem \
  --client-key=/home/work/_app/k8s/kubernetes/ssl/kube-proxy-client-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-proxy.kubeconfig
 
kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-proxy \
  --kubeconfig=kube-proxy.kubeconfig
 
kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
##### env.sh文件内容结束
#####参数说明：BOOTSTRAP_TOKEN使用在创建 TLS Bootstrapping Token 生成的token_value，KUBE_APISERVER为master节点

#### 执行脚本
cd /home/work/_app/k8s/kubernetes/cfg
sh env.sh

### 2、创建 kubelet 配置文件
vim /home/work/_app/k8s/kubernetes/cfg/kubelet.config
##### kubelet.config文件内容开始
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
address: 10.0.0.100
port: 10250
readOnlyPort: 10255
cgroupDriver: cgroupfs
clusterDNS: ["10.244.0.1"]
clusterDomain: cluster.local.
failSwapOn: false
authentication:
  anonymous:
    enabled: true
##### kubelet.config文件内容结束
#####参数说明address为master节点IP

#### 创建 /home/work/_app/k8s/kubernetes/cfg/kubelet
vim /home/work/_app/k8s/kubernetes/cfg/kubelet
##### kubelet文件内容开始
KUBELET_OPTS="--logtostderr=true \
--v=4 \
--hostname-override=10.0.0.100 \
--kubeconfig=/home/work/_app/k8s/kubernetes/cfg/kubelet.kubeconfig \
--bootstrap-kubeconfig=/home/work/_app/k8s/kubernetes/cfg/bootstrap.kubeconfig \
--config=/home/work/_app/k8s/kubernetes/cfg/kubelet.config \
--cert-dir=/home/work/_app/k8s/kubernetes/ssl_cert \
--pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google-containers/pause-amd64:3.0"
##### kubelet文件内容结束
##### 参数说明hostname-override为master节点IP，当 kubelet 启动时，如果通过 --kubeconfig 指定的文件不存在，则通过 --bootstrap-kubeconfig 指定的 bootstrap kubeconfig 用于从API服务器请求客户端证书。在通过 kubelet 批准证书请求时，引用生成的密钥和证书将放在 --cert-dir 目录中。

#### 将 kubelet-bootstrap 用户绑定到系统集群角色
kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --user=kubelet-bootstrap

#### 创建 kubelet 系统启动文件
vim /lib/systemd/system/kubelet.service
##### kubelet.service文件内容开始
[Unit]
Description=Kubernetes Kubelet
After=docker.service
Requires=docker.service
 
[Service]
EnvironmentFile=/home/work/_app/k8s/kubernetes/cfg/kubelet
ExecStart=/home/work/_app/k8s/kubernetes/bin/kubelet $KUBELET_OPTS
Restart=on-failure
KillMode=process
 
[Install]
WantedBy=multi-user.target
##### kubelet.service文件内容结束

#### 启动 kubelet 服务
systemctl daemon-reload && systemctl enable kubelet && systemctl start kubelet

#### 查看 kubelet 服务运行状态
systemctl status kubelet

```

- step13:批准 Master 加入集群
```shell
### 1、查看 CSR 列表
kubectl get csr  ###记录下name为csr_name

### 2、批准加入集群
kubectl certificate approve csr_name

### 3、验证 Master 是否加入集群
kubectl get csr

```

- step14: 部署 kube-proxy 组件
```shell

### 1、创建 kube-proxy 参数配置文件
vim /home/work/_app/k8s/kubernetes/cfg/kube-proxy
##### kube-proxy文件内容开始
KUBE_PROXY_OPTS="--logtostderr=true \
--v=4 \
--hostname-override=10.0.0.100 \
--cluster-cidr=10.244.0.0/16 \
--kubeconfig=/home/work/_app/k8s/kubernetes/cfg/kube-proxy.kubeconfig"
##### kube-proxy文件内容结束
###### 参数说明：--hostname-override在不同的节点处，要换成节点的IP

### 2、创建 kube-proxy 系统启动文件
vim /lib/systemd/system/kube-proxy.service
##### kube-proxy.service文件内容开始
[Unit]
Description=Kubernetes Proxy
After=network.target
 
[Service]
EnvironmentFile=-/home/work/_app/k8s/kubernetes/cfg/kube-proxy
ExecStart=/home/work/_app/k8s/kubernetes/bin/kube-proxy $KUBE_PROXY_OPTS
Restart=on-failure
 
[Install]
WantedBy=multi-user.target 
##### kube-proxy.service文件内容结束

### 3、启动 kube-proxy 服务、检查 kube-proxy 服务状态
#### 启动 kube-proxy 服务
systemctl daemon-reload && systemctl enable kube-proxy &&  systemctl start kube-proxy

#### 检查 kube-proxy 服务状态
systemctl status kube-proxy

```

- step14:验证 Server 服务
```shell
kubectl get cs,nodes
```

#### 备注说明
以上操作均为单节点部署操作，详细参照地址https://www.cnblogs.com/lion.net/p/10408512.html

ansible安装参考 https://github.com/easzlab/kubeasz