## Docker

### 网络原理说明
docker0作为一个网桥（网关）处于容器veth(eth0)与容器veth(eth0)以及容器veth(eth0)和宿主机网卡之间
#### 网络模式
- Host：无独立IP，和宿主机共用一个网络ns，和宿主机同IP
- Container：和关联的一个已经存在的容器共享一个网络ns
- None：关闭容器网络功能。（在只进行写磁盘的批处理等场景可用）
- Bridge：NAT模式，容器拥有独立的网络ns，并连接到docker0虚拟网卡上，通过docker0网桥和iptables nat表配置与宿主机通信(docker inspect container_name中的NETWORKS信息即为容器的网络信息)

### CNI（container network interface）
- overlay network：建立在现有网络上的虚拟网络
- BGP：边界网关协议，用于管理边缘路由器间数据包的路由方式。
- VXLAN：覆盖网络协议，可运行在现有网络之上，通过在UDP包中封装第二层以太网帧来帮助实现大型云部署
#### Flannel
flannel实际上是一种配置在第三层的覆盖网络（overlay network）就是将TCP数据包装在另一个网络包中进行路由转发和通信，让集群中每个节点都有一个子网且节点的docker容器都具有一个全集群唯一的虚拟IP。（docker0每个节点的子网网段可能相同）
- 跨主机网络路径：ContainerA eth0 -> Node1 docker0 -> Node1 flannel0(将请求封成UDP包，并通过ETCD查到目的容器所属节点的地址) -> Node2 flannel0(将请求UDP包解包并转发到docker0) -> Node2 docker0 -> ContainerB eth0
- ETCD：存储并维护了一张节点间路由表
- 优点：易安装和配置
#### Calico
Calico是配置在第三层网络采用BGP（边界网关协议）路由协议在主机之间路由数据包而无需再打包数据包。
- 优点：性能快（节点间路由无需再打包数据包）、异常排查（无需打包数据包，排查简单）、网络功能（设置负载等，与lstio继承）
#### Canal
Flannel与Calico的组合，网络层使用Flannel的覆盖网络，网络规则、流量控制使用Calico
#### Weave
在集群中每个节点中创建网状overlay network，通过各节点上的路由组件维护可用网络最新视图，当需要将流量发送到位于不同节点上的pod时weave路由组件会自动决定是采用快速数据路径发送还是回退到sleeve分组转发的方式
- 独有优点：对整个网络简单加密

### 分布式系统
分布式的出现是为了用多个机器协作完成单台机器无法完成的任务，分布式系统是由一组通过网络进行通信、为了完成共同的任务而协作的节点组成的系统
#### 分布式系统的特性
- 可扩展性：
- 高性能：
- 高可用：
- 一致性：
### CAP定理（CAP theorem）
又称布鲁尔定理，即在一个分布式计算系统中不可能同时满足一致性（Consistency 所有节点在同一时间具有相同数据）、可用性（Availability 保证每个请求不管成功或者失败都有响应）、分割容忍（Partition Tolerance 系统中任意信息的丢失或失败不影响系统的继续运作）
#### 理论核心
CAP的理论核心是一个分布式系统不可能同时满足一致性、可用性、分割容忍，最多只能同时满足其中两个
- CA：单点集群，满足一致性、可用性的系统，可扩展性不太强大
- CP：满足一致性、分区容忍性的系统，通常性能不是很高
- AP：满足可用性、分区容忍性的系统，通常对一致性要求低些

### RPC
一个节点请求另一个节点的服务
#### 组成
- 客户端（client）：服务消费方
- 客户端存根（client stub）：存放服务端地址信息，将客户端请求参数数据信息打包成网络消息，再通过网络传输发送给服务端
- 服务端存根（server stub）：将客户端传来的消息进行解包并调用本地服务处理
- 服务端（server）：服务真正的处理者
- Network Service：底层传输，可以是tcp或http
### REST
以业务为导向，将业务对象上的操作映射到http的动词，格式简单。

### Docker安装
- 1. 清除之前安装的
```
sudo apt-get remove docker docker-engine docker.io
```
- 2. 更新系统库
```
sudo apt-get update
```
- 3. 安装curl等工具
```
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```
- 4. 添加镜像源key
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
```
- 5. 添加镜像源
```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
```
- 6. 安装docker-ce
```
sudo apt-get install docker-ce
```
- 7. 再安装docker-compose
```
sudo apt-get install docker-compose
```

**打印镜像，过滤指定镜像，过滤掉最新2个镜像外，只打印镜像ID,逐个删除镜像** 
```Shell
docker images | grep 'Name' | tail -n +3 | awk '{print $3}' | xargs docker rmi
```

**Docker**
* [ ] Docker pull imageName    拉取镜像
* [ ] docker run imageName 创建容器并启动（-t 启动一个伪终端，-I打开容器的标准输入，-d后台运行，-p指定映射的IP和端口号，--run标记表示创建前若已存在则删除重建，—link name:alias连接另外一个名为name的容器连接的别名为alias）
* [ ] docker images 列出本地镜像
* [ ] docker commit 提交镜像
* [ ] docker push 上传本地镜像
* [ ] docker load 加载本地镜像
* [ ] docker build    创建镜像（-t指定tag）
* [ ] docker rmi 移除指定的本地镜像
* [ ] docker rm 移除容器（镜像对应的所有容器）
* [ ] docker start启动一个已经终止的容器
* [ ] docker stop终止容器
* [ ] docker restart 重启一个运行态的容器
* [ ] docker search 查询官方库中的响应镜像并通过docker pull进行拉取
* [ ] docker tag imageName/imageID [registryhost] [username/]imageName:tagName 将镜像打标签
* [ ] docker logs 查看日志信息
* [ ] docker inspect containerName 查看容器信息
* [ ] docker port 查看绑定的IP和端口
* [ ] docker save -o fileName imageName:version 保存指定镜像到指定文件
* [ ] docker load --input fileName 从文件中恢复镜像到本地库
* [ ] docker export container_id > fileNmae 将容器快照导出
* [ ] cat fileNmae | docker import - imageName 从指定文件中恢复镜像 或者直接docker import file

```
Dockerfile创建镜像的文件（每条指令都穿件一个镜像，然后最后一个创建完成后会将前面创建的镜像删除，层数最多为127层，创建配置文件后使用docker build命令即可创建镜像）
FROM 指令为说明镜像创建的基础
```
**注释标识**
* [ ] RUN <command> 以run开头的命令会在创建中运行，如apt-get等命令
* [ ] MAINRTAINER 指定维护者信息
* [ ] CMD [command,param1,param2,…]|command param1 param2
* [ ] EXPOSE <port> 指定容器对外暴露的端口
* [ ] ENV <key><value> 指定环境变量
* [ ] ADD source target将指定的文件复制到目标位置
* [ ] COPY source target 将本地主机文件复制到目标主机
* [ ] ENTRYPOINT [command,param1,param2]|command param1 param2 容器启动后执行的命令
* [ ] VOLUMN [“/data”] 创建挂载点
* [ ] USER daemon 指定容器运行时的用户名或uid，同run指定的一样
* [ ] WORKDIR path 为后期的run，cmd，entrypoint等命令指定工作目录
* [ ] ONBUILD [command] 指定当创建的镜像为其他镜像的基础镜像时执行的命令
```
docker-compose.yml（docker compose使用的主模版文件）
docker-compose [options] [command]
Options-f 指定特定的compose模版，默认为docker-compose.yml文件
```
**command**
* [ ] build    构建或重新构建服务
* [ ] help 获取一个命令的帮助内容
* [ ] kill 停止一个服务容器
* [ ] logs 查看日志
* [ ] port 查看容器的端口
* [ ] ps 列出所有容器
* [ ] pull 拉取服务镜像
* [ ] rm 删除停止的服务容器
* [ ] run 在一个服务上执行一个命令
* [ ] scale 设置同一个服务运行的容器个数 docker-compose scale serviceName=count
* [ ] start 启动一个已存在的服务容器
* [ ] stop 停止一个运行的服务容器
* [ ]up 构建

**文件配置说明**
* [ ] image    指定镜像名称或镜像ID
* [ ] build    指定dockerfile文件的路径
* [ ] command    容器启动后默认执行的命令
* [ ] link    连接到其服务的容器 实际值为 service:alias
* [ ] external_links    链接到外部容器
* [ ] ports 暴露端口信息（ip:port）
* [ ] expose    暴露端口
* [ ] volums    卷挂载路径设置
* [ ] environment    设置环境变量
* [ ] env_file    从文件中获取环境变量
* [ ] extends    基于已有的服务进行拓展

================================================================================
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

### Docker基础命令
* docker login 登录docker
* docker logout 退出登录
* docker search imageName 搜索指定镜像名的镜像
* docker pull ImageName:Version 拉取指定版本镜像
* docker commit containerID tagName 提交容器更改到镜像
* docker tag sourcetag newtag 打标签
* docker push tagName 将自己的镜像推送到仓库
* docker build -t tagName -f DOCKERFILEPath contextPath 创建镜像，指定镜像tag和DOCKERFILE文件路径以及上下文路径，若未指定DOCKERFILE的路径则会查找上下文路径下的DOCKERFILE文件（镜像打包方式会将上下文路径下的所有文件都传到docker服务端进行打包操作，所以指定上下文时需保证上下文路径下都是打包需要用到的文件，COPY等文件操作时也需要保证需要操作的文件处于上下文路径下）
* docker save imageName fileName 将指定镜像打包成文件
* docker load -i fileName 加载指定镜像文件
* docker run ImageName -i让容器标注输入打开 -t分配一个伪终端 -d后台运行 -p newPort:originPort指定映射出的端口 -v newVolumns:originVolimns指定卷 --mount source=volumnName|dirPath，target=filePath(in container) --name containerNmae 指定启动容器的名字
* docker start containerName 启动一个已经停止了的容器
* docker logs [containerID|NAMES] 查看一个容器的日志
* docker stop containerName 终止一个容器
* docker exec -it containerNmae bash 使用bash进入容器内部
* docker attach container 进入容器（但exit来退出时容器也会终止）
* docker import originFile tagName 从指定文件或者地址导入镜像为指定标签
* docker rm containerName 删除指定容器
* docker rmi imageName 删除置顶镜像（删除前确认是否还有由镜像启动的容器，也需要删除）
* docker volumn create volumnName 创建一个数据卷
* docker volumn inspect volumnName 查看一个数据卷信息
* docker volumn rm volumnName 移除一个数据卷

### DOCKERFILE文件规则
* FROM baseOSName 指定基础镜像
* MAINTAINER maintainerInfo 维护人员信息
* RUN cmd 执行命令行命令（1.shell方式：直接执行shell命令，2.exec方式：RUN [可执行文件,参数1,参数2]），可以多个命令用&&符链接
* COPY [--chown=<user>:<group>] originFilePaths aimPath 将上下文下的文件拷贝到镜像中的指定目录（originFilePath支持正则匹配GO语言的filepath.Match）
* ADD [--chown=<user>:<group>] origin aimPath 拷贝指定文件到指定目录下（支持origin为网络地址，且所有网络地址下载的文件权限都为600，若指定文件为tar文件压缩格式为gizp、bzip2及xz时，ADD命令会自动解压改文件到指定目录下）
* CMD <命令>|["可执行文件","参数1","参数2"] 执行命令，一般用作指定启动主进程命令
* ENTRYPOINT "<CMD>" 执行程序入口（主进程），当指定ENTRYPOINT后CMD命令即会被作为参数传入到ENTRYPOINT命令中，docker启动时也可以通过指定--entrypoint参数来指定程序入口，程序需为绝对路径！
* ENV <key> <value>|key1=value1 key2=value2 指定环境变量，RUN命令以及镜像内应用都可以使用这些环境变量
* ARG key1=value1 key2=value2 指定构建参数，同环境变量一样，只是指定的环境变量在容器中不可以使用
* VOLUME ["<路径1>","<路径2>"...]|<路径> 定义匿名卷，所有向匿名卷写入的内容都不会保存在容器的存储层，docker容器启动时可以指定挂载来替代，-v aimPath:VolumePath这样将容器中的目录指定到镜像外的目录（永久保存）
* EXPOSE <端口1>|[<端口2>...] 声明开放端口（仅仅声明，实际端口开放需要另外操作）做容器服务端口的映射到宿主机可以在启动容器时指定 -p 宿主机端口:容器提供服务器端口
* WORKDIR <工作目录路径> 指定工作目录即为指定当前所处目录
* USER <用户名>[:<用户组>] 指定当前用户


#### 注意事项
RUN、CMD、ENTRYPOINT的区别
RUN 为系统安装软件
CMD 镜像启动时执行的指令，会被docker run中的指令替换
ENTRYPOINT  指定程序入口，指令和路径需为全路径

**注意事项**
1、RUN、CMD、ENTRYPOINT的区别
   RUN 为系统安装软件、CMD 镜像启动时执行的指令，会被docker run中的指令替换、ENTRYPOINT  指定程序入口，指令和路径需为全路径