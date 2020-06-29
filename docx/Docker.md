**docker安装**
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

**指定yaml文件后台启动镜像**
```Shell
docker-compose -f office_compose.yaml up -d
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