## Doc
Visit [here](http://doc.nexusphp.org/)

# 宝塔面板安装NexusPHP1.6成功步骤

宝塔Nginx 1.18.0 + MySQL 5.6.50 + PHP-7.2 + redis 安装要点

安装准备
安装如下php扩展
redis扩展缓存器，需重启PHP-7.2
gmp扩展

开启被禁用php函数
putenv()
proc_open


以下必须在 /www/wwwroot/192.168.1.118 下ssh执行：

_git clone -b v1.6.0-beta3  https://github.com/xiaomlove/nexusphp.git ./  克隆v1.6.0-beta3移动到根目录_     **开发版代码 内容提交会出错,不建议使用**

*推荐发布版代码*

wget https://github.com/xiaomlove/nexusphp/archive/v1.6.0-beta3.zip

unzip v1.6.0-beta3.zip  解压缩,

mv -R  nexusphp-1.6.0-beta3   /www/wwwroot/192.168.1.118  移动到根目录


composer install，安装依赖

composer update，更新可能会过时的依赖关系

cp -R nexus/Install/install public/，复制 nexus/Install/install 到 public/，保证最后 public/install/install.php 存在

chmod -R 0777 /www/wwwroot/192.168.1.118，设置根目录赋予 0777 权限
以上准备工作做完，打开网站ip/域名正常会跳转安装界面。


有问题可查阅安装日志：/tmp/nexus_install_20210305.log
安装日志包含敏感数据，不要泄露。

完成后，为安全起见，请删除以下 /www/wwwroot/192.168.1.118/public/install 目录。



宝塔nginx 站点配置如下（可能需重启nginx）
```

server {
    listen 80;
    listen [::]:80;
    server_name 192.168.1.118;
    index index.php index.html index.htm default.php default.htm default.html;
    # 以实际为准
    root /www/wwwroot/192.168.1.118/public;

    #PHP-INFO-START  PHP引用配置，可以注释或修改
    include enable-php-72.conf;
    #PHP-INFO-END
    
    location / {
        index index.html index.php;
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ \.php {
        # 以实际为准
        fastcgi_pass 127.0.0.1:9000; 
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    access_log  /www/wwwlogs/192.168.1.118.log;
    error_log  /www/wwwlogs/192.168.1.118.error.log;
}

```


完整的配置
```
server
{
    listen 80;
    listen [::]:80;
    server_name 192.168.1.118;
    index index.php index.html index.htm default.php default.htm default.html;
    root /www/wwwroot/192.168.1.118/public;
    
    #SSL-START SSL相关配置，请勿删除或修改下一行带注释的404规则
    #error_page 404/404.html;
    #SSL-END
    
    #ERROR-PAGE-START  错误页配置，可以注释、删除或修改
    #error_page 404 /404.html;
    #error_page 502 /502.html;
    #ERROR-PAGE-END
    
    #PHP-INFO-START  PHP引用配置，可以注释或修改
    include enable-php-72.conf;
    #PHP-INFO-END
    
    #REWRITE-START URL重写规则引用,修改后将导致面板设置的伪静态规则失效
    include /www/server/panel/vhost/rewrite/192.168.1.118.conf;
    #REWRITE-END

    location / {
        index index.html index.php;
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ \.php {
        # 以实际为准
        fastcgi_pass 127.0.0.1:9000; 
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
    #禁止访问的文件或目录
    location ~ ^/(\.user.ini|\.htaccess|\.git|\.svn|\.project|LICENSE|README.md)
    {
        return 404;
    }
    
    #一键申请SSL证书验证目录相关设置
    location ~ \.well-known{
        allow all;
    }
    
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
    {
        expires      30d;
        error_log /dev/null;
        access_log off;
    }
    
    location ~ .*\.(js|css)?$
    {
        expires      12h;
        error_log /dev/null;
        access_log off; 
    }
    access_log  /www/wwwlogs/192.168.1.118.log;
    error_log  /www/wwwlogs/192.168.1.118.error.log;
}
```
