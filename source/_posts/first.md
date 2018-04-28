---
title: 平時小問題解決
date: 2018-04-28 11:18:40
tags:
---
## 安装matplotlib python-tk 网上解决方法需要重新编译 python matplotlib error python Tkinter module not found on Ubuntu 
> 解决方案
``` bash
sudo apt-get install python3.6-tk
```

> [stackoverflow  链接](https://stackoverflow.com/questions/6084416/tkinter-module-not-found-on-ubuntu)



## docker 镜像加速

``` bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://etsgrm2s.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## ubuntu-18.04 安装 mysql-5.7 root 密码

``` bash
sudo apt install mysql-server mysql-client

# 查询 默认 用户名 密码
sudo vi /etc/mysql/debian.cnf

# 登录 mysql 修改 root 密码
update mysql.user set authentication_string=password('password') where user='root';

# 修改远程连接
update user set host = '%' where user = 'root';

# 修改验证方式
UPDATE user SET plugin='mysql_native_password' WHERE User='root';


## option 2

mysql> CREATE USER 'YOUR_SYSTEM_USER'@'localhost' IDENTIFIED BY '';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'YOUR_SYSTEM_USER'@'localhost';
mysql> UPDATE user SET plugin='auth_socket' WHERE User='YOUR_SYSTEM_USER';
mysql> FLUSH PRIVILEGES;
```