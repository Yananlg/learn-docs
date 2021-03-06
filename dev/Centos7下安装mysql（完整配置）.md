# Centos7下安装mysql（完整配置）



**1. 下载并安装MySQL官方的Yum Repository**

```
[root@localhost ~]# wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm1
```

**使用上面的命令直接安装Yum Repository**

```
[root@localhost ~]# yum -y install mysql57-community-release-el7-10.noarch.rpm1
```

**安装MySQL服务器**

```
root@localhost ~]# yum -y install mysql-community-server1
```

**2. MySQL数据库设置** 
启动MySQL

```
[root@localhost ~]# systemctl start  mysqld.service1
```

查看MySQL运行状态

```
[root@localhost ~]# systemctl status mysqld.service1
```

此时MySQL已经开始正常运行，需要找出root的密码

```
[root@localhost ~]# grep "password" /var/log/mysqld.log1
```

如下命令登录mysql

```
# mysql -uroot -p1
```

输入初始密码，此时不能做任何事情，因为MYSQL默认必须修改密码才能正常使用

```
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new password';

# 这里会遇到一个问题，新密码设置过于简单会报错123
```

可通过如下命令查看完整的初始密码规则

```
mysql> show variables like 'validate_password';

mysql> SHOW VARIABLES LIKE 'validate_password%';
```

可通过如下命令修改

```bash
MySQL5：
mysql> set global validate_password_policy=0;
mysql> set global validate_password_length=1;
MySQL8：
mysql> set global validate_password.length=1;
mysql> set global validate_password.policy=0;
```

还有一个问题就是Yum Repository,以后每次 yum 操作都会自动更新，需要把这个卸载掉

```
[root@localhost ~]# yum -y remove mysql57-community-release-el7-10.noarch1
```

远程登录数据库出现下面出错信息 
ERROR 2003 (HY000): Can’t connect to MySQL server on ‘xxx.xxx.xxx.xxx’, 
原因是没有授予相应的权限

```
#任何主机
mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;

#指定主机
mysql>GRANT ALL PRIVILEGES ON *.* TO 'jack'@’10.10.50.127’ IDENTIFIED BY '654321' WITH GRANT OPTION;

# 然后刷新权限
mysql>flush privileges;
```

修改mysql数据库总的user表使相的用户能从某一主机登录

```
mysql> mysql -u root -p;
mysql> use mysql;

# 授权
mysql> GRANT ALL ON *.* TO 'root'@'%';

# 如显示（ERROR 1410 (42000): You are not allowed to create a user with GRANT
）没有该用户需要先创建用户
mysql> CREATE USER 'root'@'%' IDENTIFIED BY '111111';

# 然后刷新权限
mysql> flush privileges;

# 在 mysql 数据库的 user 表中查看当前 root 用户的相关信息
mysql> select host, user, authentication_string, plugin from user; 
```

客户端提供MYSQL的环境，但是不支持中文,通过以下命令可以查看mysql的字符集（MySQL8无需修改）

```
mysql>show variables like 'character_set%';
```

显示如下：

![image](https://note.youdao.com/yws/public/resource/3603e7790bd5457e32e7db9b4b934ebf/xmlnote/79CB4D7450C948AC8DE3744F5CF8020D/2946)

为了让 MySQL支持中文，需要把字符集改成UTF-8，方法如下

```
# vim /etc/mycnf1
```

改成如下内容

```
[client]
port=3306
socket=/var/lib/mysql/mysql.sock
default-character-set=utf8

[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
character-set-server=utf8

[mysql]
no-auto-rehash
default-character-set=utf8

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid1234567891011121314151617181920
```

重启mysql服务

```
# service mysql restart1
```

重新查看数据库编码

```
show variables like 'character_set%';1
```

效果如下,可看到都改为utf-8

![image](https://note.youdao.com/yws/public/resource/3603e7790bd5457e32e7db9b4b934ebf/xmlnote/5E490146F76D411F8D23B541F1EE6581/2977)