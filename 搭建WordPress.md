# 搭建WordPress

## CentOS 8

源地址：

https://help.aliyun.com/document_detail/173042.html?spm=a2c4g.11186623.6.1305.32626a94a0cjil

需要root环境

国内建议修改yum源

### 配置WordPress数据库

进入MySQL数据库。

使用`root`用户登录MySQL，并输入密码。密码为您在搭建环境时为数据库设置的密码。

```shell
mysql -uroot -p
```

为WordPress网站创建数据库。

本教程中数据库名为wordpress。

```mysql
create database wordpress;
```

创建一个新用户管理WordPress库，提高安全性。

MySQL在5.7版本后默认安装了密码强度验证插件validate_password。您可以登录MySQL后查看密码强度规则。

```mysql
show variables like "%password%";
```

创建新用户，新用户密码。

```mysql
create user 'user'@'localhost' identified by 'password';
```

赋予用户对数据库wordpress的全部权限。

```mysql
grant all privileges on wordpress.* to 'user'@'localhost';
```

使配置生效。

```mysql
flush privileges;
```

退出MySQL。

```mysql
exit;
```

### 安装WordPress

下载并解压WordPress，然后移动至网站根目录。

进入Nginx网站根目录，下载WordPress。

```shell
cd /usr/share/nginx/html
wget https://wordpress.org/wordpress-5.4.2.zip
```

**说明**： 下载过慢或失败时，可以尝试运行命令

```shell
wget http://cn.wp101.net/latest-zh_CN.zip
```

下载WordPress（`cn.wp101.net`为WordPress简体中文网站，通过该命令下载的WordPress默认为中文版本）。同时您需要注意，后续操作中压缩包的名称必须替换为`latest-zh_CN.zip`。

解压WordPress压缩包。

```shell
unzip wordpress-5.4.2.zip
```

将WordPress安装目录下的`wp-config-sample.php`文件复制到`wp-config.php`文件中，并将`wp-config-sample.php`文件作为备份。

```shell
cd /usr/share/nginx/html/wordpress
cp wp-config-sample.php wp-config.php
```

编辑`wp-config.php`文件。

```shell
vim wp-config.php
```

按i键切换至编辑模式，根据配置完成的wordpress数据库信息，修改MySQL相关配置信息，修改代码如下所示。

WordPress网站的数据信息将通过数据库的`user`用户保存在wordpress库中。

```
// ** MySQL 设置 - 具体信息来自您正在使用的主机 ** //
/** WordPress数据库的名称 */
define('DB_NAME', 'wordpress');

/** MySQL数据库用户名 */
define('DB_USER', 'user');

/** MySQL数据库密码 */
define('DB_PASSWORD', 'PASSword123.');

/** MySQL主机 */
define('DB_HOST', 'localhost');
```

修改完成后，按下Esc键后，输入`:wq`并回车，保存退出配置文件。

### 修改Nginx配置文件

运行以下命令打开Nginx配置文件。

```
vi /etc/nginx/conf.d/default.conf
```

按i键进入编辑模式。

在`location /`大括号内，将`root`后的内容替换为wordpress根目录。本示例中根目录为/usr/share/nginx/html/wordpress。[![nginx](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/zh-CN/6500430061/p167509.png)](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/zh-CN/6500430061/p167509.png)在`location ~ .php$`大括号内，将`root`后的内容替换为wordpress根目录。[![nginx](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/zh-CN/6500430061/p167510.png)](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/zh-CN/6500430061/p167510.png)修改完成后按Esc键，输入`:wq`保存并退出配置文件。

运行以下命令重启Nginx服务。

```bash
systemctl restart nginx
```

### 登录WordPress网站

域名相关和常见问题请到阿里云的网站