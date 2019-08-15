## Nginx配置文件Nginx.conf的整体结构

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190523145950269.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190523150451655.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

## 笔记
```conf
####################    全局块     ###############################
####################   作用     ###############################
#           配置影响Nginx进程的全局配置信息
#           一般包含：Nginx进程pid文件存放地址           #pid        logs/nginx.pid;
#                    Nginx日志文件存放地址及相关配置     #error_log  logs/error.log;  级别以此为：debug|info|notice|warn|error|crit|alert|emerg
#                   配置文件引入
#                   允许生成的work_processes数量        worker_processes  1; 
#
##########################################################
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;



####################    events块     ###############################
####################   作用     ###############################
#          配置影响nginx服务器或与用户的网络连接
#           一般包含：有每个进程的最大连接数             worker_connections  1024
#                    选取哪种事件驱动模型处理连接请求    use epoll;      #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport   
#                    是否允许同时接受多个网路连接        multi_accept on
#                    开启多个网络连接序列化等            accept_mutex on
#
##########################################################
events {
    worker_connections  1024;
}



####################    Http块     ###############################
####################   作用     ###############################
#          可以嵌套多个server，配置代理，缓存，引入第三方模块等
##########################################################
http {

    ####################    Http全局配置块     ###############################
    include       mime.types;#文件扩展名与文件类型映射表
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;#允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
    #sendfile_max_chunk 100k;  #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;#设置连接超时时间

    ###############文件压缩###########################
    #gzip  on;
    #文件最小压缩长度，小于则不用压缩
    #gzip_min_length 1;
    #压缩等级
    #gzip_comp_level 2;
    #要压缩的文件类型
    #gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/gif image/png


    ####################    server块     ###############################
    ####################    作用     ###############################
    #                       配置虚拟主机的相关参数
    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;


        ####################    location块     ###############################
        ####################   作用     ###############################
        #           配置请求的路由和路径以及各种页面处理情况
        ##########################################################
        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

## 反向代理配置
对于一个本地的html文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190813112122414.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)s
```java

```