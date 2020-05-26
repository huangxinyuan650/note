#### Redis
##### 持久化
原因：服务器宕机等因素会导致内存数据丢失
###### RDB（快照）
- 原理：将当前内存中的数据写入磁盘（fork一个子进程执行BGSAVE命令将内存数据写入临时文件，同时主进程继续接收客户端请求，快照生成完成替换持久化文件）。ps：持久化过程中的内容并未写入持久化文件，可能造成这段时间数据的丢失；SAVE命令会阻塞redis进程，但速度一般较快。
- 设置：save Nseconds Mcommond，每n秒执行M次写入操作，可设置多个save参数，持久化文件名、位置均可在配置文件中设置。
- 优点：1、文件紧凑、小 2、适用容灾 3、性能较优 4、大数据集恢复速度较aof快
- 缺点：1、会丢失最后一次持久化后的数据 2、大数据集时fork子进程比较耗时
##### AOF（追加）
- 原理：将所有写指令追加到持久化文件末尾，重启时重读文件执行写指令
- 设置：appendonly设置yes开启，appendfsync设置持久化频率（always：所有写指令都持久化一次；everysec（默认）：每秒钟执行一次；no：由操作系统决定，快但不安全）。（BGREWRITE）重写机制当文件越来越大时很多指令重复或者最后删除则持久化文件中只保留最后一条写操作或者剔除掉最后删掉的那条记录的命令从而使文件变小（no-appendfsync-on-rewrite no ：重写期间主进程是否继续将日志从缓冲区刷新到旧日志文件 
auto-aof-rewrite-percentage 100 文件重写需满足的增长率
auto-aof-rewrite-min-size 64mb 文件重写需满足的大小 需同时满足增长率和文件大小才触发重写机制）；redis-check-aof --fix检查aof文件，遇到第一个错误即删除之后的所有操作。
- 优点：1、耐久（持久化不影响性能） 2、只有追加 3、文件过大时可以重写  4、持久化文件可读性（未重写前可以恢复数据至想要的位置）
- 缺点：1、文件体积大于rdb 2、速度可能差于rdb 3、可能重新载入时数据不一致
##### Redis集群
###### redis集群搭建
- 1、下载redis包http://download.redis.io/releases/redis-5.0.8.tar.gz
- 2、将redis包解压至需要存放目录（/opt/redis）
- 3、编译redis，cd /opt/redis && make (前提是系统已安装好make和gcc，apt-get install make gcc -y)，确认最后未报错
- 4、根据需要开放的端口在redis文件夹下创建文件夹 mkdir 6501 6502 6503 6504 6505 6506
- 5、修改/opt/redis/redis.conf文件（为每个redis实例创建一个文件，并拷贝到上一步创建的目录下）
```
bind 0.0.0.0	 #设置可以连接终端IP
protected-mode no 	#关闭保护模式，暂时不设置密码
port 6501 	#为每个实例改一个端口
daemonize yes 	# 以守护进程的方式启动实例
pidfile /var/run/redis_6501.pid  	# 设置pid的文件，这里加上端口识别
dbfilename dump6501.rdb 	# 设置rdb持久化文件名
logfile "6501.log"      # 设置后台日志文件名
dir /opt/redis/6501  	# 生成的持久化文件及集群相关生成的文件的目录
appendonly yes	#开启aof持久化
appendfilename "appendonly6501.aof"	# aof持久化文件名
cluster-enabled yes		# 开启集群模式
cluster-config-file nodes6501.conf 		# 集群生成的的配置文件名
```
- 6、启动redis实例，/opt/redis/src/redis-server /opt/redis/6501/redis.conf (其他端口对应实例启用方式相同)
- 7、设置集群 /opt/redis/src/redis-cli --cluster create IP:6501 IP:6502 IP:6503 IP:6504 IP:6505 IP:6506 --cluster-replicas 1 (设置三主三从的redis集群)
- 8.验证，/opt/redis/src/redis-cli -h IP -p 6501 登录redis，然后查看info

###### redis集群常用操作
- redis-cli --cluster create IP:Port ... --cluster-replicas 1 创建集群
- redis-cli --cluster add-node newIP:newPort exitingIP:exittingPort --cluster-slave --cluster-master-id masterID 集群增加节点（可指定master）
- redis-cli --cluster del-node IP:Port 集群删除节点（从节点和没有slot的主节点）
- redis-cli --cluster fix IP:Port 修复节点
- redis-cli --cluster reshard IP:Port --cluster-from OriginIP:Port --cluster-to TargetIP:Port --cluster-slots SlotCount 将指定数量的slot从源实例迁移至目标实例
- redis-cli --cluster info IP:Port 查看集群信息
- redis-cli --cluster check IP:Port 检查集群
- redis-cli --cluster reblance IP:Port --cluster-weight Node1=weight1 ... Noden=weightn 平衡slot数量
- redis-cli --cluster call IP:Port ... command param1 ... 在指定节点上执行命令
- redis-cli --cluster import IP:Port 将外部数据导入集群

###### 集群管理操作（集群内操作）
- cluster Meet IP Port 将指定实例增加进集群
- cluster forget NodeID 从集群中移除指定节点（所有节点都需要执行）
- cluster replicate NodeID 将当前节点设置为指定节点的从节点
- cluster nodes 查看集群节点信息
- cluster slaves NodeID 查看指定节点的所有slave 

###### 集群节点迁移原理
Master实例迁移：需先将原Master的slave挂到新的master，然后Master实例的slot迁移至新Master，删除原master
Slave实例迁移：直接将新Slave实例挂载到Master上，然后去掉原Slave的挂载即可