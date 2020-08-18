#### 说明
Mysql主主互备即为两个mysql的互为备份机

##### Windows下安装步骤(Linux下步骤类似，基本就是装上mysql，然后修改配置来完成主从的设置)
- step1、下载mysql的zip包（目前测试版本为5.7.28不带debug的包）并解压两次，文件夹改名为master和slave，要安装两台机器或者一台机器用不同的端口装两个实例
- step2、在mster和slave文件夹下新建数据库配置文件my.ini（linux则直接在安装完成后修改my.conf配置文件），配置说明见文末，具体特殊参数设置请自行查询调整设置。
- step3、以管理员身份运行cmd，并cd到master和slave下的bin目录下（windows下切盘只需输入"盘符:"即可）
- step4、都执行安装前初始化命令完成初始化操作 mysqld --initialize --user=mysql --console 执行完成可以看到输出中生成了一个零时密码root@localhost:后端的一串字符，记录下来
- step5、执行服务安装命令 mysqld --install 服务名称（master和slave） --defaults-file="my.ini的全路径"  可看到提示服务安装完成 
- step6、启动服务 net start master|slave(刚刚定义的服务名称)
- step7、登录mysql mysql -uroot -p [-P] [-h]密码为安装时生成的零时密码，登录完成后修改mysql密码，set password=password("密码")
- step8、登录数据库后执行show master status;记住File和Position的值
- step9、登录从库（两台互为从库），设置各自的主库 change master to master_host='主库的IP',master_port=主库的端口,master_user='登录主库用户名',master_password='登录主库的密码',master_log_file='主库中的File值',master_log_pos=主库中的Position值;
- step10、启动slave 从库中执行start slave; 可以新建库和表来测试是否已经自动完成同步

##### 备注
参考文档：https://www.cnblogs.com/yeya/p/11878009.html?utm_source=gold_browser_extension

***
#### master1
```
[client]
# 端口号，默认是3306，同一机器下不同的mysql实例端口号不能相同，不同机器可以直接不写即使用3306是端口
port=3307
default-character-set=utf8

[mysqld] 
#主库配置（主库配置和备份配置主主时都需要配置，主备时对应配置一项即可）
server_id=1 ###主备的server_id需不同
log_bin=master-bin  ###主主备份时必填，slave通过此文件进行同步
log_bin-index=master-bin.index

####master写入二进制配置
###binlog-do-db=test  指定需要写入二进制文件的库
binlog-ignore-db=mysql,information_schema,performance_schema ###指定不写入二进制文件中进行同步的库
auto-increment-increment=2 ###id自增步长
auto-increment-offset=1 ###id自增起始ID

####备份配置
relay-log-index=slave-relay-bin.index
relay-log=slave-relay-bin   ###必填，备份时根据的二进制
log-slave-updates=ON ### 主主时需设置为ON

####slave同步配置
###replicate-do-db=test 指定需要同步的库
replicate-ignore-db=mysql,information_schema,performance_schema  ### 备份过滤不同步的库


# 设置为自己MYSQL的安装目录
basedir=E:/software/mysql/master
# 设置为MYSQL的数据目录，data文件夹由mysql自动生成
datadir=E:/software/mysql/master/data
port=3307
character_set_server=utf8
sql_mode=NO_ENGINE_SUBSTITUTION,NO_AUTO_CREATE_USER ###sql查询时的参数，也可安装后重新设置

# 开启查询缓存
explicit_defaults_for_timestamp=true
```
***
#### master2
```
[client]
# 端口号，默认是3306，同一个环境下不同的mysql实例端口号不能相同
port=3308
default-character-set=utf8

[mysqld] 
#主库配置
server_id=2
log_bin=master-bin
log_bin-index=master-bin.index

####master写入二进制配置
###binlog-do-db=test  指定需要写入二进制文件的库
binlog-ignore-db=mysql,information_schema,performance_schema
auto-increment-increment=2
auto-increment-offset=2

####slave同步配置
###replicate-do-db=test 指定需要同步的库
replicate-ignore-db=mysql,information_schema,performance_schema


###从库配置
relay-log-index=slave-relay-bin.index
relay-log=slave-relay-bin
log-slave-updates=ON



# 设置为自己MYSQL的安装目录
basedir=E:/software/mysql/slave
# 设置为MYSQL的数据目录，data文件夹由mysql自动生成
datadir=E:/software/mysql/slave/data
port=3308
character_set_server=utf8
sql_mode=NO_ENGINE_SUBSTITUTION,NO_AUTO_CREATE_USER

# 开启查询缓存
explicit_defaults_for_timestamp=true
```

#### windows下双活方案
- 场景：两台WindowsServer上分别安装了两个mysql数据库（主主互备）
- 说明：在其中一台机器上安装mysql-router，配置destinations（mysql server列表）为两台WindowsServer主机，模式为read-write，即可做到访问mysql-router主机路由到两个mysql上，原则上访问的为第一台mysql，当第一台挂掉可自动切换到第二台上，从而实现双活
- 安装说明：
- step1:当在两台数据库服务器上安装时直接下载windows版本（https://cdn.mysql.com//Downloads/MySQL-Router/mysql-router-8.0.18-winx64.zip），解压到指定安装目录（建议在非C盘下）
- step2：拷贝文末内容（mysqlrouter.conf）到解压目录下，生成配置文件mysqlrouter.conf，根据实际情况调整配置具体路径和数据库服务器的IP及端口
- step3:将mysql-router的bin目录加到系统环境变量或者cmd下切换到mysql-router的bin目录下，然后执行命令“mysqlrouter -c mysqlrouter.conf文件的路径”，即可启动mysql-router
- step4:应用服务（configmap.yaml）的mysql配置只需配置为mysql-router的所在机器的IP及设定的端口即可
- 备注说明：mysql-router的具体安装地址建议直接安装在应用服务器上，可以保证数据库服务器中任意一台挂掉不影响服务



```shell mysqlrouter.conf
[DEFAULT]
# 日志路径
logging_folder=E:\software\mysql-router\log
 
# 插件路径
plugin_folder=E:\software\mysql-router\lib
 
# 配置路径
config_folder=E:\software\mysql-router
 
# 运行时状态路径
#runtime_folder=E:\software\mysql-router\run
 
# 数据文件路径
data_folder=E:\software\mysql-router\data
 
[logger]
# 日志级别
level=INFO
 
# 以下选项可用于路由标识的策略部分
[routing:test]
# Router地址
bind_address=0.0.0.0:3309
# Router端口
#bind_port=3309
# 读写模式
mode=read-write
# 目标服务器
destinations=127.0.0.1:3307,127.0.0.1:3308

```

#### 安装
```shell
### Ubuntu
apt-get install mysql-server-5.7 mysql-client-5.7 -y 

### CentOS
# 下载安装包
wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
# 安装mysql
rpm -ivh mysql57-community-release-el7-11.noarch.rpm
yum install mysql-community-server
# 查看mysql服务是否启动
systemctl status mysqld.service
# 启动mysql服务（未启动时）
systemctl start mysqld.service
# 获取root初始密码
grep "password" /var/log/mysqld.log  # 获取默认密码

# 登录数据库，设置密码
mysql> set password for 'root'@'localhost'="your_password";
```
***
#### 设置登录失败策略
* 适用版本：MySql 5.7.17以上
* 方式：安装Connection-Control插件
* 步骤:
    * 1. 插件安装(登录进入mysql控制终端后执行```INSTALL PLUGIN CONNECTION_CONTROL
  SONAME 'connection_control.so';
INSTALL PLUGIN CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS
  SONAME 'connection_control.so';```)
    * 2. 修改配置（在mysqld的配置中添加: ```connection_control_failed_connections_threshold=失败次数
  connection_control_min_connection_delay=锁定后延迟登录时间（单位为毫秒) connection_control_max_connection_delay=最大延迟时间（登录锁定后每尝试一次延迟时间就会加一次锁定后延迟时间，增加次参数比较恶意导致无法访问）```）
    * 3. 重启mysql服务（```systemctl restart mysql.service```）

***
#### 备份(gzip压缩)
```shell
/usr/bin/mysqldump --lock-tables=false --single-transaction --ignore-table=db.table1 --ignore-table=db.table2  -h$HOST -u$USER -p$PASSWORD $DBNAME | gzip > $DIR/$DBNAME-$NOW.sql.gz
echo "output:  $DIR/$DBNAME-$NOW.sql.gz "

```


### 索引

#### 优点
- 大大减少了服务器需要扫描的数据量
- 帮助服务器避免排序和临时表
- 可将随机I/O变为顺序I/O

#### 主键索引（聚簇索引）
叶子节点存放主键对应行的数据（不需回表）

#### 普通索引（非聚簇索引、二级索引）
叶子节点只存对应主键的值（即通过条件查询得到主键的值，然后通过主键的值再去主键索引中查询所对应行的数据 回表）

#### 覆盖索引
所使用索引树的叶子结点中包含所需查询的字段就不需要回表

#### 联合索引
多列索引（使用覆盖索引时可在select 中直接增加联合索引包含的字段来减少回表）

#### 哈希索引
基于哈希表实现，只有针对所有索引的精确匹配才有效（根据每行的所有索引列计算一个哈希值）
缺点：1、不存行值必须回表 2、哈希值无序存储所以无法排序 3、不支持部分列索引 4、哈希冲突维护代价较高

#### 索引可匹配说明
全匹配（按序全匹配）、匹配最左前缀（索引第一列）、匹配列前缀（第一列以*开头）、匹配范围值（第一列范围匹配）、精确匹配某一列并范围匹配另一列、只访问索引查询

#### 最左匹配原则
从索引的最左边的列开始查找，当遇到范围查找时停止匹配（Mysql的查询器会针对where子句进行优化自动满足最左匹配原则）

### B/B+树
- B树（B-树）：所有节点都存索引和所在行的值，会导致树高度变高
- B+树：非叶子节点只存索引，叶子结点存索引和所在行的值（右子树含父节点）

### 索引数据结构分析
- 数组：查询、删除、修改时速度较快，但插入时需要开辟内存、数据移动影响性能
- HashTable：增删改查速度都快，但在进行模糊范围查询时需要遍历性能较慢
- 二叉树：树较高增加磁盘IO次数直接影响性能

### 事务
#### 四大特性
- 原子性（Atomicity），事物要么全做要么全不做（回滚时使用undolog）
- 一致性（Consistency），事物前后总体一致（即不会出现数据莫名改变、一般由业务保证）
- 隔离性（Isolation），事物间相互隔离，脏读（读了未提交的事物中修改的值，读已提交）、不可重复读（分别读了事物提交前后的数据值不一致，可重复读（读的时候版本不一致可用undolog恢复））、幻读（分别读了事物提交前后的数据条数不一致，可串行化）
- 持久性（Durability），改变为永久改变（redolog（先写redolog再写缓存，这样突然宕机也可以保证数据持久性））

### 锁
#### 行锁（row lock）和表锁（table lock）
只有当查询的字段是主键或者加了索引并且是指定匹配才触发行锁，其余情况除了未查询到结果不使用锁剩下的都是触发表锁
```
select * from table_name where  field = 1 for update
```

### 性能优化
#### 单条查询剖析（show profile）
- 开启profiling   set profiling=1  （会话级别）
- 执行查询语句
- 查看profiles信息show profiles; 得到的列表列举了执行的语句，找到sql对应的query_id
- 查看profile对应的具体详情  show profile for query QUERY_ID;

##### Tips
 * [ ] 指向记录的指针长度决定了表最大的记录数，mysql支持8字节，MyISAM默认6字节（256TB）
 * [ ] MyISAM表有三个文件（索引文件-MYI、表结构文件-frm、数据文件-MYD）、表级锁