原理：http://cn.linux.vbird.org/linux_server/0330nfs.php

## NFS服务搭建
在Ubuntu系统上安装并配置NFS客户机/服务器。客户端系统的NFS服务器和安装上的共享目录。
作为存储服务器的节点需要安装NFS Server服务; 其他节点作为NFS Client端挂载共享目录即可。
### 服务器配置
```
Server端: 10.1.8.107
Client端: 10.1.8.104, 10.1.8.105
```

### 设置Server端的NFS服务
```
# 安装依赖 
apt install nfs-kernel-server
# 创建目录
mkdir -p /hcmdata/nfs/hcm/
mkdir -p /hcmdata/nfs/hcm/hcm_core_document/
mkdir -p /hcmdata/nfs/hcm/photo/
mkdir -p /hcmdata/nfs/hcm/template/
mkdir -p /hcmdata/nfs/hcm/work_space/
mkdir -p /hcmdata/nfs/redis/
mkdir -p /hcmdata/nfs/redis_background/
# 更改目录权限
chown nobody:nogroup /hcmdata/nfs
# 配置nfs
vim /etc/exports
# 不开放root权限
/hcmdata/nfs/hcm    *(rw,sync,no_root_squash,no_subtree_check)
/hcmdata/nfs/redis    *(rw,sync,no_root_squash,no_subtree_check)
/hcmdata/nfs/redis_background    *(rw,sync,no_root_squash,no_subtree_check)
# 导出共享目录
exportfs -a
# 查看共享目录
exportfs -v
# 重启nfs
systemctl restart nfs.server.service
```

### 配置Client端的NFS
服务器端设置完成后，登录到客户系统，需要在Client端安装相关依赖包，并挂载共享目录。
注意：多台机器共享的情况，在每台机器都需要进行以下操作

```
# 安装NFS客户端
sudo apt-get install nfs-common
# 查看Server端可挂载的共享目录
showmount -e $SERVER_IP
# 创建挂载点
sudo mkdir /target
# 挂载远程NFS共享目录:
sudo mount $SERVER_IP:/hcmdata/nfs/hcm  /target
```


#### 验证安装目录
执行df -h命令(或者mount)可以看到挂载成功的情况,类似：
```
 sudo df -h
Filesystem                    Size  Used Avail Use% Mounted on
/dev/sda1                      20G  2.8G   16G  16% /
udev                          371M  4.0K  371M   1% /dev
tmpfs                         152M  812K  151M   1% /run
none                          5.0M     0  5.0M   0% /run/lock
none                          378M  8.0K  378M   1% /run/shm
/dev/sr0                       32M   32M     0 100% /media/CDROM
/dev/sr1                      702M  702M     0 100% /media/Ubuntu 12.04 LTS i386
10.1.8.107:/hcmdata/nfs/hcm   20G  2.8G   16G  16% /hcmdata/nfs/hcm
```


## 安全
 - 修改/etc/exports限制IP以及配置目录权限
/hcmdata/nfs/hcm    10.91.170.14(rw,sync,no_subtree_check,no_root_squash) 10.23.0.0/16(rw,sync,no_subtree_check,no_root_squash) 172.23.0.0/16(rw,sync,no_subtree_check,no_root_squash)
/hcmdata/nfs/redis    10.91.170.14(rw,sync,no_subtree_check,no_root_squash) 10.23.0.0/16(rw,sync,no_subtree_check,no_root_squash) 172.23.0.0/16(rw,sync,no_subtree_check,no_root_squash)
/hcmdata/nfs/redis_background    10.91.170.14(rw,sync,no_subtree_check,no_root_squash) 10.23.0.0/16(rw,sync,no_subtree_check,no_root_squash) 172.23.0.0/16(rw,sync,no_subtree_check,no_root_squash)
/hcmdata/nfs/es    10.91.170.14(rw,sync,no_subtree_check,no_root_squash) 10.23.0.0/16(rw,sync,no_subtree_check,no_root_squash) 172.23.0.0/16(rw,sync,no_subtree_check,no_root_squash)

 - 使用iptables限制portmap服务（用于RPC服务）的连接
#!/bin/sh
add(){
    iptables -I INPUT -p TCP -s 10.23.0.0/16 --dport 111 -j ACCEPT
    iptables -I INPUT -p UDP -s 10.23.0.0/16 --dport 111 -j ACCEPT
    iptables -I INPUT -p TCP -s 172.23.0.0/16 --dport 111 -j ACCEPT
    iptables -I INPUT -p UDP -s 172.23.0.0/16 --dport 111 -j ACCEPT
    iptables -A INPUT -p TCP -s 0.0.0.0/0 --dport 111 -j DROP
    iptables -A INPUT -p UDP -s 0.0.0.0/0 --dport 111 -j DROP
}
delete(){
    iptables -D INPUT -p TCP -s 10.23.0.0/16 --dport 111 -j ACCEPT
    iptables -D INPUT -p UDP -s 10.23.0.0/16 --dport 111 -j ACCEPT
    iptables -D INPUT -p TCP -s 172.23.0.0/16 --dport 111 -j ACCEPT
    iptables -D INPUT -p UDP -s 172.23.0.0/16 --dport 111 -j ACCEPT
    iptables -D INPUT -p TCP -s 0.0.0.0/0 --dport 111 -j DROP
    iptables -D INPUT -p UDP -s 0.0.0.0/0 --dport 111 -j DROP
}

check(){
    echo "----   showmount result -----"
    showmount -e 10.6.224.16
    echo "----   rpcinfo result -----"
    rpcinfo -p 10.6.224.16
}

check

add

echo "--------------------------------"

check


验证方法：执行showmount -e IP 和 rpcinfo -p IP 进行查看是否有输出
