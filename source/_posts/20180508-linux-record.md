---
title: Code 日常
date: 2018-05-08 10:07:55
tags: linux python docker
categories: bug
---

## 日常问题、小技巧收集

主要记录日常遇到的问题及解决方法
包括但不限于 python/linux/docker/database

## linux命令终端显示-bash-4.2#解决方法

* 近期在折腾docker centos 使用别人的镜像 发现终端显示 `bash-4.2#` 很是忧伤（之前遇到，好记性不如烂笔头）

    ``` bash
    # 这里的root 可以替换成你的用户
    cp /etc/skel/.bashrc /root/  
    cp /etc/skel/.bash_profile  /root/  

    ```

* 然后 exit 重新登录就好了

## linux 文件编辑

* 在线上操作文件 一般都是 vi 但是 vi 适用于各种花式操作 这里介绍实用的简单的操作

    ```bash

    # echo > file 覆盖文件
    echo "" > file  # 这个操作可以快速清空某一文件
    echo "Hello World" > file
    # echo >> file 追加文件
    echo "Hello World" >> file


    # cat 配合 EOF > 覆盖  >> 追加
    cat << EOF > file
    > Hello
    > Hello world
    ```

    EOF是END Of File的缩写,表示自定义终止符.既然自定义,那么EOF就不是固定的,可以随意设置别名,在linux按ctrl-d就代表EOF
    **EOF一般会配合cat能够多行文本输出.**

## pyhon matplotlib 安装

* 问题： 安装matplotlib python-tk 网上解决方法需要重新编译 python matplotlib error python Tkinter module not found on Ubuntu
* 解决方案

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

## python 相关bug

1. python 换源以及格式化pip输出

    ```bash

    cd ~ & mkdir .pip
    vi ~/.pip/pip.conf

    [global]
    timeout = 60
    index-url = http://pypi.douban.com/simple
    trusted-host = pypi.douban.com
    format=columns
    ```

2. 在运行 tensorboard 出错

    原因分析：mxnet 要求 numpy 的版本与tensorboard不匹配
    `mxnet 1.4.1 has requirement numpy<1.15.0,>=1.8.2, but you'll have numpy 1.16.3 which is incompatible.`

    ```bash
    # 错误详情
    ➜  tensorflow git:(master) ✗ tensorboard --logdir ./
    ModuleNotFoundError: No module named 'numpy.core._multiarray_umath'
    ImportError: numpy.core.multiarray failed to import

    The above exception was the direct cause of the following exception:

    Traceback (most recent call last):
    File "<frozen importlib._bootstrap>", line 968, in _find_and_load
    SystemError: <class '_frozen_importlib._ModuleLockManager'> returned a result with an error set
    ImportError: numpy.core._multiarray_umath failed to import
    ImportError: numpy.core.umath failed to import
    2019-06-03 12:42:23.607527: F tensorflow/python/lib/core/bfloat16.cc:675] Check failed: PyBfloat16_Type.tp_base != nullptr
    [1]    79828 abort      tensorboard --logdir ./

    # 解决
    pip3 install -U numpy

    # 启动
    tensorboard --logdir  ./
    ```
