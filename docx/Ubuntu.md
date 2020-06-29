#### 所需基础软件
* 装vim、net-tools apt-get install vim，net-tools
* 设置可以ssh
openssh-server apt-get install openssh-server
* 设置可以远程root登录
修改/etc/ssh/sshd_config文件中的PermitRootLogin为yes
ssh相关公钥信息位置 ~/.ssh/id_rsa.pub
* 网络工具ping等
apt-get install iputils-ping
* 虚拟机新装CentOS无法上网问题，可能网络未启用（修改/etc/sysconfig/network-scripts/下对应设备配置的onboot为yes后重启即可）

### Ubuntu18下docker安装
* step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
* step 2: 安装GPG证书
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
* Step 3: 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
* Step 4: 更新并安装 Docker-CE
sudo apt-get -y update
sudo apt-get -y install docker-ce

### 其他版本Ubuntu下安装Docker
1. 清除之前安装的
sudo apt-get remove docker docker-engine docker.io
2. 更新系统库
sudo apt-get update
3. 安装curl等工具
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
4. 添加镜像源key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
5. 添加镜像源
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
6. 安装docker-ce
sudo apt-get install docker-ce
7. 再安装docker-compose
sudo apt-get install docker-compose

### 安装nfs
1. server端安装
sudo apt install nfs-kernel-server
配置共享目录，编辑需要共享的目录
vim /etc/exports
添加 目录（绝对路径） 
[ip(支持网段统配)] (rw,no_root_squash,no_subtree_check,sync,no_wdelay,insecure)
服务重启    service nfs-server restart

2. client端安装
sudo apt install nfs-common
查找指定机器上放出可挂载的路径
showmount -e IP
挂载远程目录到本地指定目录
mount -t nfs IP:sourcePath  targetPath
卸载挂载的目录
umount targetPath
