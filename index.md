# nginx.GitHub.io

View the page[GO](https://hong-chen-0.github.io/nginx.GitHub.io/)

nginx说明

```
########### 每个指令必须有分号结束。#################
#user administrator administrators;  #配置用户或者组，默认为nobody nobody。
#worker_processes 2;  #允许生成的进程数，默认为1
#pid /nginx/pid/nginx.pid;   #指定nginx进程运行文件存放地址
error_log log/error.log debug;  #制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别以此为：debug|info|notice|warn|error|crit|alert|emerg
events {
    accept_mutex on;   #设置网路连接序列化，防止惊群现象发生，默认为on
    multi_accept on;  #设置一个进程是否同时接受多个网络连接，默认为off
    #use epoll;      #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
    worker_connections  1024;    #最大连接数，默认为512
}
http {
    include       mime.types;   #文件扩展名与文件类型映射表
    default_type  application/octet-stream; #默认文件类型，默认为text/plain
    #access_log off; #取消服务日志    
    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式
    access_log log/access.log myFormat;  #combined为日志格式的默认值
    sendfile on;   #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
    sendfile_max_chunk 100k;  #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    keepalive_timeout 65;  #连接超时时间，默认为75s，可以在http，server，location块。

    upstream mysvr {   
      server 127.0.0.1:7878;
      server 192.168.10.121:3333 backup;  #热备
    }
    error_page 404 https://www.baidu.com; #错误页
    server {
        keepalive_requests 120; #单连接请求上限次数。
        listen       4545;   #监听端口
        server_name  127.0.0.1;   #监听地址       
        location  ~*^.+$ {       #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
           #root path;  #根目录
           #index vv.txt;  #设置默认页
           proxy_pass  http://mysvr;  #请求转向mysvr 定义的服务器列表
           deny 127.0.0.1;  #拒绝的ip
           allow 172.18.5.54; #允许的ip           
        } 
    }
}
```
```
#user  nobody;

#==工作进程数，一般设置为cpu核心数
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {

    #==最大连接数，一般设置为cpu*2048
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    
    #==客户端链接超时时间
    keepalive_timeout  65;

    #gzip  on;

    #当配置多个server节点时，默认server names的缓存区大小就不够了，需要手动设置大一点
    server_names_hash_bucket_size 512;

    #server表示虚拟主机可以理解为一个站点，可以配置多个server节点搭建多个站点
    #每一个请求进来确定使用哪个server由server_name确定
    server {
        #站点监听端口
        listen       8800;
        #站点访问域名
        server_name  localhost;
        
        #编码格式，避免url参数乱码
        charset utf-8;

        #access_log  logs/host.access.log  main;

        #location用来匹配同一域名下多个URI的访问规则
        #比如动态资源如何跳转，静态资源如何跳转等
        #location后面跟着的/代表匹配规则
        location / {
            #站点根目录，可以是相对路径，也可以使绝对路径
            root   html;
            #默认主页
            index  index.html index.htm;
            
            #转发后端站点地址，一般用于做软负载，轮询后端服务器
            #proxy_pass http://10.11.12.237:8080;

            #拒绝请求，返回403，一般用于某些目录禁止访问
            #deny all;
            
            #允许请求
            #allow all;
            
            add_header 'Access-Control-Allow-Origin' '*';
            add_header 'Access-Control-Allow-Credentials' 'true';
            add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
            add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
            #重新定义或者添加发往后端服务器的请求头
            #给请求头中添加客户请求主机名
            proxy_set_header Host $host;
            #给请求头中添加客户端IP
            proxy_set_header X-Real-IP $remote_addr;
            #将$remote_addr变量值添加在客户端“X-Forwarded-For”请求头的后面，并以逗号分隔。 如果客户端请求未携带“X-Forwarded-For”请求头，$proxy_add_x_forwarded_for变量值将与$remote_addr变量相同  
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            #给请求头中添加客户端的Cookie
            proxy_set_header Cookie $http_cookie;
            #将使用代理服务器的主域名和端口号来替换。如果端口是80，可以不加。
            proxy_redirect off;
            
            #浏览器对 Cookie 有很多限制，如果 Cookie 的 Domain 部分与当前页面的 Domain 不匹配就无法写入。
            #所以如果请求 A 域名，服务器 proxy_pass 到 B 域名，然后 B 服务器输出 Domian=B 的 Cookie，
            #前端的页面依然停留在 A 域名上，于是浏览器就无法将 Cookie 写入。
            
　　         #不仅是域名，浏览器对 Path 也有限制。我们经常会 proxy_pass 到目标服务器的某个 Path 下，
            #不把这个 Path 暴露给浏览器。这时候如果目标服务器的 Cookie 写死了 Path 也会出现 Cookie 无法写入的问题。
            
            #设置“Set-Cookie”响应头中的domain属性的替换文本，其值可以为一个字符串、正则表达式的模式或一个引用的变量
            #转发后端服务器如果需要Cookie则需要将cookie domain也进行转换，否则前端域名与后端域名不一致cookie就会无法存取
　　　　　　  #配置规则：proxy_cookie_domain serverDomain(后端服务器域) nginxDomain(nginx服务器域)
            proxy_cookie_domain localhost .testcaigou800.com;
            
            #取消当前配置级别的所有proxy_cookie_domain指令
            #proxy_cookie_domain off;
            #与后端服务器建立连接的超时时间。一般不可能大于75秒；
            proxy_connect_timeout 30;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
    
　　#当需要对同一端口监听多个域名时，使用如下配置，端口相同域名不同，server_name也可以使用正则进行配置
　　#但要注意server过多需要手动扩大server_names_hash_bucket_size缓存区大小
　　server {
　　　　listen 80;
　　　　server_name www.abc.com;
　　　　charset utf-8;
　　　　location / {
　　　　　　proxy_pass http://localhost:10001;
　　　　}
　　}
　　server {
　　　　listen 80;
　　　　server_name aaa.abc.com;
　　　　charset utf-8;
　　　　location / {
　　　　　　proxy_pass http://localhost:20002;
　　　　}
　　}
}
```
```
1.$remote_addr 与 $http_x_forwarded_for 用以记录客户端的ip地址；
2.$remote_user ：用来记录客户端用户名称；
3.$time_local ： 用来记录访问时间与时区；
4.$request ： 用来记录请求的url与http协议；
5.$status ： 用来记录请求状态；成功是200；
6.$body_bytes_s ent ：记录发送给客户端文件主体内容大小；
7.$http_referer ：用来记录从那个页面链接访问过来的；
8.$http_user_agent ：记录客户端浏览器的相关信息；
```
