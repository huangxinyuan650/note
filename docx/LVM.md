### 中铁建LVM扩容

`fdisk -l`看磁盘概况

```
fdisk /dev/xvda
```

进入后按m可以查看可用指令

```
p #打印情况
n #新建分区（利用剩余扇区，start是起始扇区，end是结束扇区）
p #选择primary类型
3 #选择/dev/xvda3
104855552 #因为/dev/xvda5结束的扇区是104855551，而且整个磁盘在其后面还有很多未使用的扇区，所以从那里开始往后分
+100G #选择往后加100G
t #设置分区类型
3 #选择给/dev/xvda3设置分区类型
8e #给/dev/xvda3设置分区类型为Linux LVM
p #再次打印详情，如无问题则按w保存，如有问题则按q退出。或者ctrl+D也可退出
w # 保存分区修改
```

设置物理卷，卷组，逻辑卷：

```

# 列出所有可用块设备（此时刚刚设置的块是不可见的）
lsblk 

# 重读分区
partprobe /dev/xvda 
#重读之后再执行lsblk可以看到新的设备了

# 创建PV 
pvcreate /dev/xvda3
# 查看PV
pvs

#  将PV加入VG
vgextend ubuntu-vg /dev/xvda3 
# 查看VG
vgs

# 将当前LV剩余空间加入到指定VG
lvextend -l +100%FREE /dev/ubuntu-vg/root 
# 查看LV
lvs
#查看文件系统，此时挂接的目录的大小还是原来的，需要通知文件系统进行更新
df -h

# 调整空间大小
resize2fs /dev/ubuntu-vg/root 

# 用df可以看到可以看到lv扩容已经生效
df -h ./ 

```