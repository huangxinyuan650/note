# GitLab community私有搭建

## 步骤
- step1: 拉取gitlab-ce镜像
```
    # 若镜像拉取较慢可通过其他方式获取并导入
    docker pull gitlab/gitlab-ce
```

- step2: 创建gitlab所使用的配置文件、log、data挂载目录
```
    mkdir -p /opt/gitlab-ce/config
    mkdir -p /opt/gitlab-ce/data
    mkdir -p /opt/gitlab-ce/logs
```

- step3: 启动gitlab镜像
启动时注意目录挂载、端口开放(开启https时将443端口映射到宿主机端口，容器内部端口无需修改，其他端口同理，若不需要开放则不需要映射出来)
```
docker run -it -d -p 443:443 -p 80:80 -p 22:22 --name gitlab --restart always  -v /opt/gitlab-ce/config:/etc/gitlab -v /opt/gitlab-ce/logs:/var/log/gitlab -v /opt/gitlab-ce/data:/var/opt/gitlab gitlab/gitlab-ce:latest
```

- step4: 开启https访问
增加external_url参数和ssl证书，进入挂载出去的目录/opt/gitlab-ce/config，修改gitlab.rb文件，增加external_url地址和开启nginx，拷贝ssl证书到/opt/gitlab-ce/config/ssl目录下
```
    cd /opt/gitlab-ce/config
    vim gitlab.rb 
    # 增加 external_url "https://code-gitlab.bigy.cn" （gitlab根据地址的协议和端口决定nginx监听的端口）
    # 增加 nginx["enable"] = true
    # 其他参数选择性开放，参照https://docs.gitlab.com/omnibus/settings/nginx.html#manually-configuring-https

    # 拷贝ssl证书到ssl文件夹下，若在nginx参数中指定证书路径则可以拷贝至相应路径下即可
    cp code-gitlab.bigy.cn.crt /opt/gitlab-ce/config/ssl
    cp code-gitlab.bigy.cn.key /opt/gitlab-ce/config/ssl

    # docker容器内重新加载配置文件
    gitlab-ctl reconfigure
```

- step5: 登录验证
浏览器访问地址https://code-gitlab.bigy.cn，若成功会有一系列的账号及密码设置等，按要求设置完成即可