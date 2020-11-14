# DockerRegistry搭建

## 部署步骤
- step1: 创建镜像仓库登录用到的账号密码文件
```
    # 安装密钥生成工具
    apt-get install apache2-utils
    # 生成密钥文件，并保存好后面将用到(用户名hxy，密码123456)
    htpasswd -Bbn hxy 123456 > /opt/auth/htpasswd
```

- step2: 拉取docker registry镜像文件
```
    # 不加版本号则拉取的为最新的版本镜像，目前暂时未发现版本引发的问题（若拉取很慢则可通过其他方式拉取后再导入）
    docker pull registry
```

- step3: 创建镜像存储挂载目录、密钥文件挂载目录、ssl证书存放挂载目录并将密钥文件拷贝至/opt/registry/auth下和将ssl证书的cert和key文件拷贝至/opt/registry/certs目录下
```
    # 目录创建
    mkdir -p /opt/registry/registry
    mkdir -p /opt/registry/auth
    mkdir -p /opt/registry/certs

    # 文件拷贝
    cp ***.cert /opt/registry/certs/    
    cp ***.key /opt/registry/certs/    
    cp htpasswd /opt/registry/auth/    
```

- step4: 启动docker registry
registry默认端口为5000，当使用ssl时可通过修改绑定端口修改监听443并映射到宿主机的443端口，当不使用ssl时则可直接将容器的5000端口映射到宿主机的某个未被使用的端口即可
```
    docker run -it -d \
        -v /opt/registry:/var/lib/registry \
        -v /opt/cert:/certs \
        -v /opt/auth:/auth \
        -e REGISTRY_AUTH=htpasswd \
        -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
        -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
        -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
        -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/1_bigy_bundle.crt \
        -e REGISTRY_HTTP_TLS_KEY=/certs/2_bigy.key  \
        -p 443:443  \
        --name registry \
        --restart always \
        registry:latest
```

- step5: 域名解析
通过修改hosts将想要作为镜像仓库的域名指向当前节点

- step6: 登录验证
```
    docker login -u hxy -p 123456
```