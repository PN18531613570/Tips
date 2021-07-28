# 安装LNMP

## CentOS 8

源地址：

https://help.aliyun.com/document_detail/173042.html?spm=a2c4g.11186623.6.1305.32626a94a0cjil

需要root环境

国内建议修改yum源

### 准备编译环境

1.关闭防火墙

查看防火墙状态

```shell
systemctl status firewalld
```

停止防火墙服务

```shell
systemctl stop firewalld
```

禁用防火墙服务

```shell
systemctl disable firewalld
```

2.关闭SELinux

查看SELinux的状态

```shell
sestatus
```

查看SELinux运行模式

```shell
getenforce
```

SELinux分为’disabled‘，‘permissive’，‘enforcing’状态

`enforcing`：强制模式。违反 SELinux 规则的行为将被**阻止**并**记录到日志中**。

`permissive`：宽容模式。违反 SELinux 规则的行为只会**记录到日志中**。一般为调试用。

`disabled`：关闭 SELinux。

SELinux运行模式切换

```shell
setenforce [0|1]
```

选项与参数:
0：转成 Permissive 宽容模式;
1：转成 Enforcing 强制模式

如永久更改请自行查询

### 安装Nginx

Nginx默认使用80端口，如80端口占用请自行解决

```shell
yum install -y nginx
```

启用Nginx服务

```shell
systemctl enable nginx
```

开启Nginx服务

```shell
systemctl start nginx
```

查看Nginx状态

```
systemctl status nginx
```

//阿里云一键安装Nginx1.16.1版本

```shell
dnf -y install http://nginx.org/packages/centos/8/x86_64/RPMS/nginx-1.16.1-1.el8.ngx.x86_64.rpm
```

查看Nginx版本

```shell
nginx -v
```



### 安装MySQL

下载rh8,centos8的rpm包

```shell
wget https://dev.mysql.com/get/mysql80-community-release-el8-1.noarch.rpm
```

安装

```shell
yum localinstall -y  ./mysql80-community-release-el8-1.noarch.rpm
```

检查是否安装成功

```shell
yum repolist enabled | grep "mysql.*-community.*"
```

安装

```shell
yum install -y mysql-community-server
```

//阿里云一键安装MySQL

```shell
dnf -y install @mysql
```

查看MySQL版本

```shell
mysql -V
```

### 安装PHP

//阿里云

添加和更新epel

```shell
dnf -y install epel-release
dnf update epel-release
```

删除无用的软件包缓存并更新软件源

```shell
dnf clean all
dnf makecache
```

启用php:7.3模块

```shell
dnf module enable php:7.3 -y
```

安装PHP相应模块

```shell
dnf install -y php php-curl php-dom php-exif php-fileinfo php-fpm php-gd php-hash php-json php-mbstring php-mysqli php-openssl php-pcre php-xml libsodium
```

查看PHP版本

```shell
php -v
```

### 配置Nginx

Nginx的配置文件地址/etc/nginx/nginx.conf

备份配置文件

```shell
cd /etc/nginx/conf.d
cp default.conf default.conf.bak
```

修改默认配置文件

```shell
vi default.conf
```

在location大括号内，修改以下内容。

```
location / {
    #将该路径替换为您的网站根目录。
    root   /usr/share/nginx/html;
    #添加默认首页信息index.php。
    index  index.html index.htm index.php;
}
```

去掉被注释的location ~ \.php$ 大括号内容前的#，并修改大括号的内容。

修改完成如下所示。

```
location ~ \.php$ {
    #将该路径替换为您的网站根目录。
    root           /usr/share/nginx/html;
    #Nginx通过unix套接字与PHP-FPM建立联系，该配置与/etc/php-fpm.d/www.conf文件内的listen配置一致。
    fastcgi_pass   unix:/run/php-fpm/www.sock;
    fastcgi_index  index.php;
    #将/scripts$fastcgi_script_name修改为$document_root$fastcgi_script_name。
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    #Nginx调用fastcgi接口处理PHP请求。
    include        fastcgi_params;
}
```

保存退出文件

启用Nginx服务（开机自启动）

```shell
systemctl enable nginx
```

开启Nginx服务

```shell
systemctl start nginx
```

### 配置MySQL

启用MySQL服务（开机自启动）

```shell
systemctl enable mysqld
```

开启MySQL服务

```shell
systemctl start mysqld
```

查看MySQL状态

```shell
systemctl status mysqld
```

执行MySQL安全性操作并设置密码

```shell
mysql_secure_installation
```

命令运行后，根据命令行提示执行如下操作。

1. 输入Y并回车开始相关配置。

2. 选择密码验证策略强度，输入

   2

   并回车。

   策略0表示低，1表示中，2表示高。建议您选择高强度的密码验证策略。

3. 设置MySQL的新密码并确认。

   本示例设置密码xxxxxxxx。

4. 输入Y并回车继续使用提供的密码。

5. 输入Y并回车移除匿名用户。

6. 设置是否允许远程连接MySQL。

   - 不需要远程连接时，输入Y并回车。
   - 需要远程连接时，输入N或其他任意非Y的按键，并回车。

7. 输入Y并回车删除`test`库以及对`test`库的访问权限。

8. 输入Y并回车重新加载授权表。

### 配置PHP

修改PHP配置文件

打开配置文件

```shell
vi /etc/php-fpm.d/www.conf
```

找到`user = apache`和`group = apache`，将`apache`修改为`nginx`

![img](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/4458755951/p130166.png)

新建phpinfo.php文件，用于展示PHP信息。

1. 运行以下命令新建文件。

   ```shell
   vim <网站根目录>/phpinfo.php  #将<网站根目录>替换为您配置的网站根目录。
   ```

   网站根目录是您在nginx.conf文件中`location ~ .php$`大括号内配置的`root`值，如下图所示。

   [![lnmp-root-dir](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/4458755951/p69633.png)](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/4458755951/p69633.png)

   本教程配置的网站根目录为/usr/share/nginx/html，因此命令为：

   ```shell
   vim /usr/share/nginx/html/phpinfo.php
   ```

2. 按i进入编辑模式。

3. 输入下列内容，函数`phpinfo()`会展示PHP的所有配置信息。

   ```php
   <?php echo phpinfo(); ?>
   ```

4. 按Esc键后，输入:wq并回车以保存并关闭配置文件。

启动PHP-FPM

```shell
systemctl start php-fpm
```

设置PHP-FPM开机自启动

```shell
systemctl enable php-fpm
```

### 测试LNMP平台

打开浏览器输入http://127.0.0.1/phpinfo.php

返回结果如下图所示，表示LNMP环境部署成功。

![phpinfo](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/4458755951/p130169.png)

测试完成后删除phpinfo.php文件

```shell
rm -rf <网站根目录>/phpinfo.php   #将<网站根目录>替换为您在nginx.conf中配置的网站根目录
```

本教程配置的网站根目录为/usr/share/nginx/html，因此命令为：

```shell
rm -rf /usr/share/nginx/html/phpinfo.php
```

