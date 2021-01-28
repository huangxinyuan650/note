### Redis
#### 为啥redis快
- 1、纯内存操作
- 2、单线程操作，避免频繁切换上下文
- 3、采用非阻塞I/O多路复用机制
#### [布隆过滤器](./Minitree.md)

#### 持久化
原因：服务器宕机等因素会导致内存数据丢失
##### RDB（快照）
- 原理：将当前内存中的数据写入磁盘（fork一个子进程执行BGSAVE命令将内存数据写入临时文件，同时主进程继续接收客户端请求，快照生成完成替换持久化文件）。ps：持久化过程中的内容并未写入持久化文件，可能造成这段时间数据的丢失；SAVE命令会阻塞redis进程，但速度一般较快。
- 设置：save Nseconds Mcommond，每n秒执行M次写入操作，可设置多个save参数，持久化文件名、位置均可在配置文件中设置。
- 优点：1、文件紧凑、小 2、适用容灾 3、性能较优 4、大数据集恢复速度较aof快
- 缺点：1、会丢失最后一次持久化后的数据 2、大数据集时fork子进程比较耗时
- 持久化命名：bgsave
##### AOF（追加）
- 原理：将所有写指令追加到持久化文件末尾，重启时重读文件执行写指令
- 设置：appendonly设置yes开启，appendfsync设置持久化频率（always：所有写指令都持久化一次；everysec（默认）：每秒钟执行一次；no：由操作系统决定，快但不安全）。（BGREWRITE）重写机制当文件越来越大时很多指令重复或者最后删除则持久化文件中只保留最后一条写操作或者剔除掉最后删掉的那条记录的命令从而使文件变小（no-appendfsync-on-rewrite no ：重写期间主进程是否继续将日志从缓冲区刷新到旧日志文件 
auto-aof-rewrite-percentage 100 文件重写需满足的增长率
auto-aof-rewrite-min-size 64mb 文件重写需满足的大小 需同时满足增长率和文件大小才触发重写机制）；redis-check-aof --fix检查aof文件，遇到第一个错误即删除之后的所有操作。
- 优点：1、耐久（持久化不影响性能） 2、只有追加 3、文件过大时可以重写  4、持久化文件可读性（未重写前可以恢复数据至想要的位置）
- 缺点：1、文件体积大于rdb 2、速度可能差于rdb 3、可能重新载入时数据不一致
- 重写命令： bgrewriteaof

#### 复制
##### 同步
###### 同步过程
- 1、从服务器向主服务器发送sync命令
- 2、主服务器执行bgsave命令生成rdb文件，并将新的写命令记录到缓冲区
- 3、主服务器将生成好的rdb文件传给从服务器
- 4、从服务器加载rdb文件更新数据至主服务器执行bgsave前的状态
- 5、主服务器将记录在积压缓冲区的写命令传给从服务器，从服务器执行接收到的写命令保持数据和主服务器当前状态一致
###### 部分重同步实现
主从复制偏移量、主服务器积压缓冲区、服务器运行ID
- 1、从服务器通过psync将自己的复制偏移量传给主服务器
- 2、主服务器判断偏移量之后的数据是否还在积压缓冲区中来确认执行部分重同步操作还是完全重同步操作

##### 命令传播
- 保证主从同步后状态持续达到一致，主服务器会将新接收到的写命令对从服务器执行命令传播操作

##### 事务（ACID）
- 原子性：事务内容要么全执行要么都不执行（不支持回滚（log日志在执行命令后才生成））
- 一致性：事务执行前后一致（即使执行出错后面的命令依然执行，单个命令执行失败未修改数据对一致性无影响）
- 隔离型：多个事务并发执行时各个事务间不产生影响（redis命令执行是串行的）
- 持久性：只有当持久化设置为aof且策略为always时才具有持久性

#### Redis集群
##### redis集群说明
- slot（槽）:2048*8=16384个槽分布在所有的master上，key->slot->node，根据key计算出所属的槽并找到对应的node然后move到对应的node进行操作
##### redis集群搭建（docker搭建见文末说明）
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
#### redis基本操作
- redis-cli -h IP keys "key*"|xargs redis-cli -h IP del 删除指定key缓存
- bgsave 手动触发rdb持久化
- barewriteaof 手动触发aof重写

##### redis集群常用操作
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

##### 集群管理操作（集群内操作）
- cluster Meet IP Port 将指定实例增加进集群
- cluster forget NodeID 从集群中移除指定节点（所有节点都需要执行）
- cluster replicate NodeID 将当前节点设置为指定节点的从节点
- cluster nodes 查看集群节点信息
- cluster slaves NodeID 查看指定节点的所有slave 

###### 集群节点迁移原理
Master实例迁移：需先将原Master的slave挂到新的master，然后Master实例的slot迁移至新Master，删除原master
Slave实例迁移：直接将新Slave实例挂载到Master上，然后去掉原Slave的挂载即可


#### Redis数据结构说明
```
// redisServer数据结构
struct redisServer {
    // 服务器中数据库数量(默认16)，由配置中的database决定
    int dbnum;
    // 一个数组保存服务器中的所有数据库
    redisDB *db;
    // 秒级精度的系统当前unix时间戳，由servercron函数每100毫秒更新一次
    time_t unixtime;
    // 毫秒级精度的系统当前uninx时间戳
    long long mstime;
    // 关闭服务器标识1:关闭服务器，0:不做动作
    int shutdowm_asap;
}

// redisDB数据结构
struct redisDB {
    // 保存一个库中所有的键值对{key1:value1,key2:value2,...}
    dict *dict;
    // 保存key的过期时间{key1:timestamp1,key2:timestamp2,...}
    // expire、expireat、pexpire最终都会转换成pexpireat命令，保存到期Unix时间戳
    dict *expires;
} redisDb;

// redisClient数据结构
struct redisClient {
    // 记录当前客户端所使用的数据库
    redisDb *db;
}
```

#### 过期键删除策略
##### 定时删除
设置键过期时间的同时创建定时器，到时立即删除（内存友好 量大对cpu不友好）
##### 惰性删除
过期不管，当获取键值时判断是否过期，若过期则删除，否则正常返回（cpu友好，当失效键多时对内存不友好）
##### 定期删除
每隔一段时间对数据库进行检查并删除其中的过期键（删除多少过期键和检查几个数据库由算法决定）
##### 内存淘汰策略 maxmemory-policy
- Noeviction：当内存不足以容纳新key时，新写入操作报错
- Allkeys-lru：当内存不足时，在键空间中移除最近最久未使用（事件）的key
- Allkeys-lfu：当内存不足时，在键空间中移除最近最少未使用（频率）的key
- Allkeys-random：当内存不足时，在键空间中随机删除某个key
- Volatile-lru：当内存不足时，在设置了过期时间键空间中删除最近最久未使用的key
- Volatile-lru：当内存不足时，在设置了过期时间键空间中删除最近最少未使用的key
- Volatile-random：当内存不足时，在设置了过期时间键空间中随机删除一个key（有更早过期的key优先删除）
- Volatile-ttl：当内存不足时，在设置了过期时间键空间中删除更早过期的key

##### Redis value数据类型（String、List、Hash、Set、ZSet）
```
value类型
typedef struct redisObject {

    // 类型(REDIS_STRING、REDIS_LIST、REDIS_HASH、REDIS_SET、REDIS_ZSET)
    unsigned type:4;

    // 编码(
    // REDIS_ENCODING_INT   long类型的整数、
    // REDIS_ENCODING_EMBSTR    embstr编码的简单动态字符串、
    // REDIS_ENCODING_RAW   简单动态字符串、
    // REDIS_ENCODING_HT    字典、
    // REDIS_ENCODING_LINKEDLIST    双端链表、
    // REDIS_ENCODING_ZIPLIST   压缩列表、
    // REDIS_ENCODING_INTSET    整数集合、
    // REDIS_ENCODING_SKIPLIST  跳跃表和字典)
    unsigned encoding:4;

    // 指向底层实现数据结构的指针
    void *ptr;
    //...
} robj;
```

##### 字符串（String）
###### 数据结构
```
struct sdshdr{
    int len; //字符串长度，buf中已使用长度
    int free;  //buf中未使用长度
    char buf[]; //存储字符串数据，总长度=len + free + 1
}
```
字符串扩展原则：小于1M，则free长度等于扩展长度，大于1M的free长度等于1M
###### 基本操作
- 设置值 set key value [EX seconds|PX milliseconds]
- 获取key的值 get key
- expire|expireat|pexpire|pexpireat key time  为指定key设置超时时间（最终都是转换为pexpireat命令）
- setnx key value 原子性的设置数据并设置过期时间，当key存在返回0，key不存在设置完成后返回1，setnx和expire配置使用做避免死锁的分布式锁

##### 列表（List）
###### 数据结构（ziplist或者linkedlist）

##### Hash（哈希）
###### 数据结构（ziplist或者hashtable）
当哈希对象保存的键值长度都小于65字节且键值对数量小于512时采用ziplist否则采用hashtable（hash-max-ziplist-value、hash-max-ziplist-entries）

##### Set（集合）
###### 数据结构（intset或者hashtable）
当集合对象保存的所有值都为整数且元素数量小于512时采用intset否则采用hashtable（set-max-intset-entries）
使用hashtable时键为元素字符串值为null

##### ZSet（有序集合）
###### 数据结构（ziplist或者skiplist）
当有序集合元素数量小于128个且元素成员长度都小于64字节采用ziplist否则采用skiplist（zset-max-ziplist-entries、zset-max-ziplist-value）
ziplist结构（\[元素1成员、元素1分值、元素2成员、元素2分值、...\]）按分值由小到大存储

##### Stream(流)
- xadd key [MAXLEN [~|=] <count>] <ID|*> [field1 value1] [field2 value2]... (设置流信息，maxlen用于裁剪流大小)
- xrange key startID endID [COUNT count] （获取流消息，startID('-'为最小)、endID('+'为最大)为消息的范围ID）
- xrevrange key endID startID [COUNT count] (获取消息，逆序返回)
- xdel key IDS (删除消息中指定ID的数据，可多个ID)
- xgroup create key groupName id-or-$ (为指定key创建一个新消费组，id为消费组开始消费的消息ID（$为最后一项ID）)
- xgroup setid key groupName id-or-$ （修改指定消费组的消费的消息last_id）
- xgroup destroy key groupName (删除指定消费组)
- xgroup delconsumer key groupName consumerName (删除指定消费组中的指定消费者)
- xreadgroup group consumer [COUNT count] [BLOCK milliseconds] streams key [key...] ID [id...] (消费消费组中的消息，ID数要与key对应,'>'为未传递给其他消费者的消息)
- xread [COUNT count] [BLOCK milliseconds] streams key [key...] ID [id...] (读取消息队列中的消息)
- xack key group ID [id...] (确认一个或多个消息，使其从待确认列表中删除)
- xpending key group [start end count] [consumer] (获取某个消费组或者消费者未确认的消息)
- xtrim key maxlen [~|=] count (裁剪消息队列的大小，～为模糊裁剪、=为精确裁剪)
- xlen key (获取消息队列的长度)
- xinfo [consumers key groupName] [groups key] [stream key] [help] （分别查询消息队列指定的group下的消费者、消费队列下的group、消息队列下的消息信息）
- 备注：xadd命令往消息队列中增加消息，然后通过xreadgroup命令消费消息，消费完成后通过xack命令确认消费完成，但以上操作均不会删除消息队列中的消息，只有在确认所有group和consumer消费完成后再执行xdel命令删除已经消费的消息
###### stream实现消息队列


#### 事件
文件事件和时间事件处理逻辑
```
def aeProcessEvents():
    // 获取到达时间离当前时间最近的时间事件
    // 计算最近时间事件到达还有多少毫秒
    // 根据最近达到时间城建timeeval结构
    // 阻塞等待文件事件产生，最大阻塞时间由timeeval结构决定
    // 若最近到达时间为0（小于0被处理成0），则等待文件事件不阻塞
    // 处理已产生的文件事件
    // 处理所有已到达的时间事件


def main():
    // 初始化服务器
    init_server();
    // 一直处理事件直到服务器关闭为止
    while server_is_not_shutwown():
        aeProcessEvents()
    // 服务器关闭操作
    clean_server()
```
##### 文件事件
服务器对套接字操作的抽象（服务器与客户端以及服务器与服务器连接），服务器通过监听并处理这些文件事件来完成一系列网络通信操作

###### 文件事件处理器
以单线程的方式使用I/O多路复用程序监听多个套接字，实现高性能的网络通信模型
- 套接字
- I/O多路复用程序：将所有产生的事件都放到一个队列中然后逐个向文件事件分派器分发
- 文件事件分派器（Dispatcher）
- 事件处理器（连接应答处理器、命令请求处理器、命令回复处理器）

备注：
Redis6版本支持的多线程只是在I/O处理（命令读取、结果应答）上增加的多线程，在文件事件分派、命令解析、命令执行等操作上仍然是单线程
##### 时间事件
服务器对给定时间点需要执行的操作的抽象（定时事件和周期性事件根据时间事件处理器返回的结果判断）

#### 初始化服务器
- step1: 初始化服务器状态结构（设置服务器运行ID、默认运行频率、默认配置文件路径、服务器运行架构、默认端口号、rdb和aof持久化条件、服务器LRU时钟、创建命令表（将list变成dict））

- step2: 载入配置项（通过配置文件和启动命令中的参数）

- step3: 初始化服务器数据结构（clients链表、db数组、pubsub_channels字典、lua环境、slowlog等）和一些重要设置（设置进程信号处理器、创建共享对象、打开监听端口、创建时间事件、打开或创建aof文件、初始化服务器后台I/O模块）

- step4: 还原数据库状态（若启用aof持久化则使用aof文件恢复否则使用rdb文件）

- step5: 执行事件循环

#### 命令请求的执行过程
- step1: 客户端发送命令请求（命令会被转换成标准的redis协议，*3(含命令的参数个数)\r\n$3(第一个参数的长度)\r\nSET\r\n$3(第二个参数的长度)\r\nKEY\r\n$4(第三个参数的长度)\r\nVALUE\r\n...（协议中以\r\n分隔））

- step2: 服务端按redis协议格式读取请求并保存到客户端输入缓冲区

- step3: 服务端对输入缓冲区的命令进行解析并分别将参数和参数个数保存到客户端状态的argv和argc属性中

- step4: 命令执行器根据argv[0]中的参数匹配commondtable中的命令并将命令保存到客户端状态的cmd属性中

- step5: 命令执行器进行执行前的命令检查、参数检查、身份认证检查、内存使用情况检查（maxmemory）、命令在当前是否允许被执行等预备操作保证命令可被正常执行

- step6: 命令执行器调用命令的实现函数完成命令执行（命令参数等都保存在客户端状态的argv参数中）并将命令回复保存到客户端状态的输出缓冲区，同时为客户端套接字关联回复处理器(结果回复将由回复处理器在客户端可写时将输出缓冲区的内容写入)

- step7: 命令执行器完成慢查询统计、命令执行耗时统计、持久化、向从库命令传播等

```
# docker创建redis集群（注意防火墙）
docker run --name redis-node1 -p 8001:8001 -p 18001:18001 -it -d  redis:6.0.5 redis-server --port 8001 --cluster-enabled yes --bind 0.0.0.0
docker run --name redis-node2 -p 8002:8002 -p 18002:18002 -it -d  redis:6.0.5 redis-server --port 8002 --cluster-enabled yes --bind 0.0.0.0
docker run --name redis-node3 -p 8003:8003 -p 18003:18003 -it -d  redis:6.0.5 redis-server --port 8003 --cluster-enabled yes --bind 0.0.0.0
docker run --name redis-node4 -p 8004:8004 -p 18004:18004 -it -d  redis:6.0.5 redis-server --port 8004 --cluster-enabled yes --bind 0.0.0.0
docker run --name redis-node5 -p 8005:8005 -p 18005:18005 -it -d  redis:6.0.5 redis-server --port 8005 --cluster-enabled yes --bind 0.0.0.0
docker run --name redis-node6 -p 8006:8006 -p 18006:18006 -it -d  redis:6.0.5 redis-server --port 8006 --cluster-enabled yes --bind 0.0.0.0

/opt/redis/src/redis-cli --cluster create IP:6501 IP:6502 IP:6503 IP:6504 IP:6505 IP:6506 --cluster-replicas 1
```