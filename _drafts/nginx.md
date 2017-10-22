# 安装

```
sudo apt-get install nginx
```
安装后的位置

+ 服务地址：/etc/init.d/nginx　

+ 配置地址：/etc/nginx/　　如：/etc/nginx/nginx.conf

+ Web默认目录：/usr/share/nginx/http/　　如：usr/share/nginx/index.html

+ 日志目录：/var/log/nginx/　　如：/var/log/nginx/access.log

+ 主程序文件：/usr/sbin/nginx

# 启动

```
sudo /etc/init.d/nginx start
sudo /etc/init.d/nginx stop
```

或者


```
nginx
nginx -s stop — fast shutdown
nginx -s quit — graceful shutdown
nginx -s reload — reloading the configuration file
nginx -s reopen — reopening the log files
```

通过`ps -ax | grep nginx`查看运行中的nginx进程


# 配置文件

`/usr/local/nginx/conf`,`/etc/nginx`,`/usr/local/etc/nginx`中的某一个文件夹下。我的在`/etc/nginx`目录下有nginx.conf文件。

假设要配置如下一个网站，有一个`/data/www`目录存放HTML文件，一个`/data/images`存放图片文件，那么需要在一个http块中设置一个包含两个location块的server块。:


按照以下步骤实施：

+ 打开cd到/etc/nginx/sites-available目录，使用sudo vim ./default 来修改该目录下的default的配置文件。

已经包括了一些`server`块的样例，当新加一个服务时启动一个新`server`块，一个server相当于一个服务器虚拟机。

```
server {
        listen 8080 default_server;
        listen [::]:8080 default_server;
        # SSL configuration
        #
        # listen 443 ssl default_server;
        # listen [::]:443 ssl default_server;


        root /var/www/html;

        # 首页配置
        index index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                try_files $uri $uri/ =404;
        }                
}
server {
        listen 80;  
        listen [::]:80;
        server_name himot;

        index index.html;
        location / {
            root /home/ding/Downloads/himot-sim;
        }

        location /images/ {
            root /data;
        }
}
```

+ 保存配置文件，输入`sudo service nginx restart`或者`sudo systemctl restart nginx.service`重启服务
