python -m SimpleHTTPServer
 python -m http.server
 
x-pack开启
elasticsearch.yml中添加xpack.security.enabled: true

ik安装
离线安装或者在线安装

卷组挂载
/usr/share/elasticsearch/data

docker run -it -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.5.2


#### Elasticsearch7镜像使用


- 1、镜像拉取
docker pull es官方地址

- 2、镜像启用
docker run -it -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" imageName

- 3、es密码设置
docker ps |grep elasticsearch找到es容器的container ID
然后 docker exec -it ContainerID bash进入容器
执行下 bin/elasticsearch-setup-passwords interactive 会确认然后要求设置密码，根据提示设置密码后 执行exit退出容器
验证方式：本地浏览器访问 http://127.0.0.1:9200会要求输入用户名和密码 用户名为elastic，密码为刚设置，正常返回es版本等信息即可


### ELK搭建
#### kibana
kibana连接es时使用--link elasticsearch:elasticsearch，且在config/kibana.yml文件中添加 elasticsearch.username和elasticsearch.password参数，其中连接es的访问地址的参数也可以在kibana.yml文件的elasticsearch.hosts: [ "http://elasticsearch:9200" ]参数中调整
注入环境变量ELASTICSEARCH_HOSTS
/usr/share/kibana/config/kibana.yml


#### logstash
/usr/share/logstash/config/logstash.yml
port 5044

#### filebeat
/usr/share/filebeat/filebeat.yml
