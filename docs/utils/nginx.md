# Nginx 基础配置

> Nginx 下载：[http://nginx.org/en/download.html](http://nginx.org/en/download.html)

## 常用配置

> 配置 SPA+SSL

```nginx
user  root;
worker_processes  1;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    include       proxy.conf;
    default_type  application/octet-stream;
    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  mall.1qishun.com;
        rewrite ^/(.*)$ "https://mall.1qishun.com/$1" permanent;
    }

        # HTTPS server
    #
    server {
        listen       443 ssl;
        server_name  mall.1qishun.com;

        ssl_certificate      ssl/web/mallpc/3182454_mall.1qishun.com.pem;
        ssl_certificate_key  ssl/web/mallpc/3182454_mall.1qishun.com.key;

        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;

        location / {
            root   /var/www/mallPC/;
            try_files $uri $uri/ /index.html;
            index  index.html index.htm;
        }
    }
}
```

## 一、安装 nginx

### 在 Linux 中安装

#### 1、安装环境

> 在安装 nginx 前首先要确认系统中安装：`gcc`、`pcre-devel`、`zlib-devel`、`openssl-devel`

```nginx
//一键安装上面四个依赖
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
```

#### 2、下载并解压压缩包

```nginx
//创建一个文件夹
cd /usr/local
mkdir nginx
cd nginx
//下载tar包
wget http://nginx.org/download/nginx-1.16.1.tar.gz
tar -xvf nginx-1.16.1.tar.gz
```

#### 3、安装 nginx

```nginx
//进入nginx目录
cd /usr/local/nginx
//执行命令
./configure
//执行make命令
make
//执行make install命令
make install
```

#### 4、启动 nginx

> 在安装目录下启动

```nginx
方法一：
/usr/local/nginx/sbin/nginx

方法二:
//进入nginx目录
cd /usr/local/nginx/sbin
//执行启动命令
./nginx
```

### 在 Windows 中安装

#### 1、下载

> 在官网中下载`nginx windows`版本后并解压  
> [Nginx 下载地址](http://nginx.org/en/download.html)

#### 2、运行

> 使用`命令行`工具进行启动，千万不要双击`nginx.exe`文件启动

#### 3、启动 nginx 服务

> 启动时会一闪而过是正常的

```nginx
start nginx
```

## 二、基础命令

### 启动命令

```nginx
// linux
/usr/local/nginx/sbin/nginx
//windows
start nginx
```

### 停止服务

#### 1、立即停止服务

```nginx
nginx  -s stop
```

#### 2、从容停止服务

```nginx
nginx  -s quit
```

#### 3、结束进程

```nginx
//linux
killall nginx
```

### 重启服务

```nginx
nginx -s reload
```

## 三、基础配置

### `nginx.conf` 文件解读

> nginx 主要模块配置

```nginx
#运行用户，可以不进行设置
#user  nobody;
#工作进程数，一般设置为cpu核心数
worker_processes  1;

#错误日志存放目录
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#进程pid存放位置
#pid        logs/nginx.pid;

events {
    #单个后台进程的最大并发数
    worker_connections  1024;
}


http {
    include       mime.types; #文件扩展名与类型映射表
    default_type  application/octet-stream; #默认文件类型

    #设置日志模式
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main; #nginx访问日志存放位置

    sendfile        on; #开启高效传输模式
    #tcp_nopush     on; #nginx访问日志存放位置

    keepalive_timeout  65; #保持连接的时间，也叫超时时间

    #gzip  on; #开启gzip压缩

    #当配置多个server节点，默认server names的缓存区大小就不够了，需要手动设置大一点server_names_hash_bucket_size 512;

    #配置多个server节点来搭建多个站点
    #每一个请求进来确定使用哪个server由server_name确定
    server {
        listen       80; #站点监听端口
        server_name  localhost; #站点访问域名

        #charset koi8-r; #编码格式，避免url参数乱码

        #access_log  logs/host.access.log  main;

        #location用来匹配同一域名下多个URI的访问规则
        #比如动态资源如何跳转，静态资源如何跳转等
        #location后面跟着的/代表匹配规则
        location / {
            root   html; #站点根目录，相对路径或者绝对路径
            index  index.html index.htm; #默认主页

            #拒绝请求，返回403，一般用于某些目录禁止访问
            #deny all;

            #允许请求
            #allow all;

            #转发后端站点地址，一般用于做软负载，轮询后端服务器
            #proxy_pass http://10.11.12.237:8080;

            #重新定义或者添加发往后端服务器的请求头
            add_header 'Access-Control-Allow-Origin' '*';

            #给请求头中添加客户请求主机名
            proxy_set_header Host $host;

            #给请求头中添加客户端IP
            proxy_set_header X-Real-IP $remote_addr;

            #将$remote_addr变量值添加在客户端“X-Forwarded-For”请求头的后面，并以逗号分隔。
            #如果客户端请求未携带“X-Forwarded-For”请求头，$proxy_add_x_forwarded_for变量值将与$remote_addr变量相同
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            #给请求头中添加客户端的Cookie
            proxy_set_header Cookie $http_cookie;

            #将使用代理服务器的主域名和端口号来替换。如果端口是80，可以不加。
            proxy_redirect off;

            #与后端服务器建立连接的超时时间。一般不可能大于75秒；
            proxy_connect_timeout 30;
        }

        #error_page  404              /404.html; # 配置404页面

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html; #错误状态码页
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

### 访问权限设置

#### 1、基础权限设置

```nginx
location / {
    allow  45.76.202.231; //允许访问
    deny   45.76.202.232; //禁止访问
}
```

#### 2、指令优先级

> 在同一个块下的两个权限指令，先出现的设置会覆盖后出现的设置（也就是谁先触发，谁起作用）

```nginx
location / {
    deny   all;
    allow  45.76.202.231; // 将会不起作用
}
```

#### 3、复杂访问控制权限匹配

> `=`号代表精确匹配，使用了`=`后是根据其后的模式进行精确匹配

```nginx
// imgs目录下文件允许访问
location =/imgs{
    allow all;
}
// admin目录下文件禁止访问
location =/admin{
    deny all;
}
```

#### 4、使用正则表达式设置访问权限

```nginx
//禁止访问php结尾的文件
location ~\.php$ {
    deny all;
}
```

### 反向代理

> 客户端请求发到代理服务器，代理服务器再把请求发送到自己设置好的内部服务器上

```nginx
server{
    listen 80;
    server_name nginx.org;
    location / {
        #代理到此服务器中
        proxy_pass http://baidu.com;
        #重新定义或者添加发往后端服务器的请求头
        add_header 'Access-Control-Allow-Origin' '*';

        #给请求头中添加客户请求主机名
        proxy_set_header Host $host;

        #给请求头中添加客户端IP
        proxy_set_header X-Real-IP $remote_addr;

        #将$remote_addr变量值添加在客户端“X-Forwarded-For”请求头的后面，并以逗号分隔。
        #如果客户端请求未携带“X-Forwarded-For”请求头，$proxy_add_x_forwarded_for变量值将与$remote_addr变量相同
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        #给请求头中添加客户端的Cookie
        proxy_set_header Cookie $http_cookie;

        #将使用代理服务器的主域名和端口号来替换。如果端口是80，可以不加。
        proxy_redirect off;

        #与后端服务器建立连接的超时时间。一般不可能大于75秒；
        proxy_connect_timeout 30;

        proxy_read_timeout 600;
        proxy_send_timeout 600;
    }
}
```

### 解决跨域

> 主域名: mysite.com  
> 请求连接: `http://mysite.com/apis/getPCBannerList.html`

```nginx
server{
    listen 80;
    server_name mysite.com;

    #请求跨域，这里约定代理请求url path是以/apis/开头
    location ^~/apis/ {
        # 这里重写了请求，将正则匹配中的第一个()中$1的path，拼接到真正的请求后面，并用break停止后续匹配
        rewrite ^/apis/(.*)$ /$1 break;
        proxy_pass https://www.kaola.com/;
    }
}
```

## 适配 PC 和移动端

```nginx
location / {
    # 移动、pc设备适配
    if ($http_user_agent ~* '(Android|webOS|iPhone|iPod|BlackBerry)') {
        rewrite ^.+ http://mysite-H5.com;
    }
}
```

## 参考资料

### [Nginx Windows 详细安装部署教程](https://www.cnblogs.com/taiyonghai/p/9402734.html)

### [Nginx 与前端开发](https://juejin.im/post/5bacbd395188255c8d0fd4b2)