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
