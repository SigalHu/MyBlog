由于微信公众平台需要的80端口已经被apache占用，所以需要基于apache配置反向代理。

代理服务器就相当于一个中介，有正向代理与反向代理两种。在正向代理中，客户端通过代理服务器访问目标服务器，代理服务器扮演着客户端的角色，真正的客户端对目标服务器不可见，比如科学上网。在反向代理中，代理服务器扮演着目标服务器的角色，例如，当客户端通过某个域名获取资源时，这些资源可能并不是从该域名绑定的服务器获取，该服务器也许只是作为代理服务器将收到的客户端请求转发给特定服务器。

本人在阿里云ECS上搭建微信公众平台服务器，由于80端口已经被apache占用，所以需要配置apache作为代理服务器接收来自微信服务器的请求，并将该请求转发给微信公众平台服务器（使用6670端口）。具体步骤如下：

**第1步：** 创建sites-available与sites-enabled目录，sites-available目录将会存放所有的虚拟主机文件，而sites-enabled目录将会存放我们想对外提供服务的主机的符号链接
```shell
mkdir /usr/local/apache/sites-available
mkdir /usr/local/apache/sites-enabled
```
**第2步：** 编辑apache的配置文件
```shell
vi /usr/local/apache/conf/httpd.conf
```
找到以下两条，把#号去掉
```
#LoadModule proxy_module modules/mod_proxy.so
#LoadModule proxy_http_module modules/mod_proxy_http.so
```
在文件末尾添加一行用以声明额外配置文件所在的可选目录
```
IncludeOptional sites-enabled/*.conf
```
**第3步：** 在sites-available目录下创建文件
```shell
vi /usr/local/apache/sites-available/web.conf
```
并添加如下内容，当微信服务器访问ServerName的80端口时，将会指向6670端口
```xml
<VirtualHost *:80>
	ServerName 此处填写你在微信公众平台上绑定的域名或IP
	ServerAlias 此处填写你在微信公众平台上绑定的域名或IP
	ProxyPass / http://127.0.0.1:6670/
	ProxyPassReverse / http://127.0.0.1:6670/
</VirtualHost>
```
**第4步：** 在sites-enabled目录下创建符号链接，**注意：此处必须使用完整路径**
```shell
ln -s /usr/local/apache/sites-available/web.conf /usr/local/apache/sites-enabled/web.conf
```
**第5步：** 重启apache
```shell
service httpd restart
```
#### 参考链接
[http://www.jianshu.com/p/b34c78bf9bf0](http://www.jianshu.com/p/b34c78bf9bf0)</br>
[http://blog.csdn.net/zhouyingge1104/article/details/44459655](http://blog.csdn.net/zhouyingge1104/article/details/44459655)
