#### 1. 安装nginx

查询nginx安装包

```shell
yum list nginx
```

![](CentOS6系统LNMP环境搭建及WordPress安装\1.PNG)

发现没有nginx的rpm包，所以需要先从[http://nginx.org/packages/centos/6/noarch/RPMS/](http://nginx.org/packages/centos/6/noarch/RPMS/)更新rpm依赖库

```shell
rpm -Uvh http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm
```

![](CentOS6系统LNMP环境搭建及WordPress安装\2.PNG)

安装nginx

```shell
yum install nginx
```

启动nginx

```shell
service nginx start
```

通过ip访问可以看到

![](CentOS6系统LNMP环境搭建及WordPress安装\3.PNG)

将nginx设置为开机自启

```shell
chkconfig nginx on
chkconfig | grep nginx
```

![](CentOS6系统LNMP环境搭建及WordPress安装\4.PNG)

#### 2. 安装mysql

查询mysql安装包

```shell
yum list mysql*
```

![](CentOS6系统LNMP环境搭建及WordPress安装\5.PNG)

mysql的版本太低，从[https://dev.mysql.com/downloads/repo/yum/](https://dev.mysql.com/downloads/repo/yum/)选择相应的rpm包

```shell
rpm -Uvh https://repo.mysql.com//mysql80-community-release-el6-1.noarch.rpm
```

![](CentOS6系统LNMP环境搭建及WordPress安装\6.PNG)

编辑repo文件，选择mysql5.7，**注意：** 不要安装mysql8.0，否则会导致wordpress安装报错

```shell
vim /etc/yum.repos.d/mysql-community.repo
```

```bash
# Enable to use MySQL 5.7
[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/6/$basearch/
enabled=1  # 设置为1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql80-community]
name=MySQL 8.0 Community Server
baseurl=http://repo.mysql.com/yum/mysql-8.0-community/el/6/$basearch/
enabled=0  # 设置为0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

安装mysql

```shell
yum install mysql-server
```

启动mysql

```shell
service mysqld start
```

将mysql设置为开机自启

```shell
chkconfig mysqld on
chkconfig | grep mysqld
```

![](CentOS6系统LNMP环境搭建及WordPress安装\7.PNG)

查看root用户默认密码

```shell
grep "password" /var/log/mysqld.log
```

登陆mysql

```shell
mysql -uroot -p
```

设置root用户密码

```sql
alter user 'root'@'localhost' identified by '新密码';
flush privileges;
```

重启mysql

```shell
service mysqld restart
```

#### 3. 安装php

查询php安装包

```shell
yum list php
```

![](CentOS6系统LNMP环境搭建及WordPress安装\8.PNG)

同样版本太低，更新rpm包

```shell
rpm -Uvh http://mirror.webtatic.com/yum/el6/latest.rpm
```

安装php以及wordpress必需的组件

```shell
yum install php70w php70w-mysql php70w-fpm
```

启动php-fpm

```shell
service php-fpm start
```

将php-fpm设置为开机自启

```shell
chkconfig php-fpm on
chkconfig | grep php-fpm
```

![](CentOS6系统LNMP环境搭建及WordPress安装\9.PNG)

#### 4. 安装并配置wordpress

从[https://cn.wordpress.org/txt-download/](https://cn.wordpress.org/txt-download/)下载wordpress

```shell
wget https://cn.wordpress.org/wordpress-4.9.4-zh_CN.tar.gz
```

解压到/home/目录下

```shell
tar -zxvf wordpress-4.9.4-zh_CN.tar.gz -C /home/
```

登陆mysql并添加数据库，命名为wordpress

```shell
mysql -uroot -p
```
```sql
 create database wordpress;
 show databases;
```

![](CentOS6系统LNMP环境搭建及WordPress安装\10.PNG)

备份nginx的配置文件

```shell
cp /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.bak
```

修改配置文件

```shell
vim /etc/nginx/conf.d/default.conf
```
```bash
server {
    listen       80;
    server_name  www.xxx.com;  # 你的域名或ip

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    root   /home/wordpress;   # wordpress根目录
    index  index.html index.htmi index.php;

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

重启nginx

```shell
service nginx restart
```

通过浏览器访问你的域名或ip

![](CentOS6系统LNMP环境搭建及WordPress安装\11.PNG)

填写mysql的用户名和密码

![](CentOS6系统LNMP环境搭建及WordPress安装\12.PNG)

报错，显示不能写入wp-config.php文件

![](CentOS6系统LNMP环境搭建及WordPress安装\13.PNG)

查看wordpress文件夹，发现是权限问题

```shell
ll /home | grep wordpress
```

![](CentOS6系统LNMP环境搭建及WordPress安装\14.PNG)

查看用户和用户组

```shell
grep -E '^(user|group)' /etc/php-fpm.d/www.conf
```

![](CentOS6系统LNMP环境搭建及WordPress安装\15.PNG)

设置访问权限

```shell
chown -R apache:apache /home/wordpress
```

![](CentOS6系统LNMP环境搭建及WordPress安装\16.PNG)

这次没有报错，安装成功

![](CentOS6系统LNMP环境搭建及WordPress安装\17.PNG)

**参考链接**

[Centos6.8 yum安装LNMP](https://www.cnblogs.com/willamwang/p/8241506.html)<br>
[CentOS6.5安装MySQL5.7详细教程](https://www.cnblogs.com/lzj0218/p/5724446.html)<br>
[Nginx+php+mysql+wordpress搭建自己的博客站点](https://blog.csdn.net/u013381397/article/details/77891947)<br>
[使用nginx利用虚拟主机搭建WordPress博客](https://yq.aliyun.com/articles/43139)<br>
[centos6.5 系统-搭建lamp（php7）环境](http://www.cnblogs.com/zsl123/p/6812735.html)<br>
[解决wordpress下载插件，安装失败，无法创建目录问题](https://blog.csdn.net/qq_32846595/article/details/54766833)<br>
[CentOS7 安装 mysql8](https://blog.csdn.net/managementandjava/article/details/80039650)<br>
[Linux(CentOS)下设置nginx开机自动启动和chkconfig管理](https://blog.csdn.net/u013870094/article/details/52463026)