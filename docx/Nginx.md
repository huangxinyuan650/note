**Ubuntu下安装方式**
直接apt-get install nginx即可（安装后的路径在/etc/nginx）
***
**基础知识**
1. Nginx命令
    * -c </path/to/config> 为 Nginx 指定一个配置文件，来代替缺省的
    * -t 不运行，而仅仅测试配置文件。nginx 将检查配置文件的语法的正确性，并尝试打开配置文件中所引用到的文件
    * -v 显示 nginx 的版本
    * -V 显示 nginx 的版本，编译器版本和配置参数
    * nginx直接启动，kill进程直接关闭，nginx -s reload重启
2. Nginx配置
    * include file/dictionary 引入文件或路径主要用作配置文件的引入，任何地方都可以用
    * pid为工作进程ID
    * user 指定Nginx Worker进程运行用户，默认是nobody帐号
    * worker_priority 分配工作进程优先级
    * alias file-path|directory-path 该指令指定用于指示位置的路径
***
**配置文件**
主配置文件为nginx.conf，可以在主配置文件中include引入其他配置文件
配置文件结构
events {}
http{
    server {}
    server {}
}

* user username group; 用户和组
* worker_processes 8; 指定衍生工作进程数（一般为core数或者两倍）
* error_log file_path level; 错误日志路径及级别
* pid file_path; 进程ID存放文件路径
* worker_rlimit_nofile 51200; 指定文件描述符数量
* events { worker_connections 555} 最大连接数
* http{}
* gzip on; 页面压缩，是浏览器浏览速度更快，nginx压缩，浏览器自动解压
* default_type application/octet-stream;默认字符集
* client_max_body_size 8M;上传文件大小限制
* location [ = | ~ | ~* | ^~ | @ ] /uri/ { … }
* 正则 location 匹配让步普通 location 的严格精确匹配结果；但覆盖普通 location 的最大前缀匹配结果

负载均衡杂：
* upstream模块是nginx负载均衡的主要模块
* server指令 用于指定后端服务器的名称和参数
upstream指令用于设置一组可以在proxy_pass和fastcgi_pass指令中使用的代理服务器，默认的负载均衡方式是轮询，upstream模块中的server用于指定后端服务器的名称和参数（名称可以是IP，域名，端口）；在upstream中配置多个实际机器地址及其权重等参数，然后在server中直接使用upstream名
* ![040ba0e01a445bb7a605cffdcb9df77e.png](evernotecid://BDE184C5-B600-4E45-9177-5AA7E4BC8425/appyinxiangcom/19063070/ENResource/p4)
* ![15bc7a1e6350961a9035702d97e065e3.png](evernotecid://BDE184C5-B600-4E45-9177-5AA7E4BC8425/appyinxiangcom/19063070/ENResource/p3)
