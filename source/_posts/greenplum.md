---
title: Greenplum
date: 2018-05-30 10:52:58
tags: 数据仓库
---

## 基本介绍

Greenplum是一个MPP（海量并行处理）计算框架的分布式数据库，其数据库引擎层是基于著名的Postgresql数据库，企业级数据库产品，现已开源。Greenplum拥有丰富的特性，包括：

1. 完全支持ANSI SQL 2008标准和SQL OLAP 2003 扩展，支持ODBC和JDBC
2. 支持分布式事务，支持ACID
3. 支持行存储、列存储，以及可通过外部表的方式访问其它关系型数据库或者Hadoop
4. 拥有良好的线性扩展能力，支持上千个节点

## 环境介绍

|     主机    |  IP |  内存(G)   |  系统  |
| :--------:  | :-----------:  |:--------:  | :----:  |
| mdw         | 172.16.16.134  |  16   |   CentOS Linux release 7.4.1708   |
| sdw1        | 172.16.16.135  | 16   |   CentOS Linux release 7.4.1708   |
| sdw1        | 172.16.16.138  |  16   |  CentOS Linux release 7.4.1708    |

## 系统参数调整

1. 修改hosts文件(三台主机)

    ``` bash
    172.16.16.134 mdw
    172.16.16.135 sdw1
    172.16.16.138 sdw2
    ```

2. 修改或添加/etc/sysctl.conf(三台主机)

    ```bash
    xfs_mount_options = rw,noatime,inode64,allocsize=16m
    kernel.shmmax = 500000000
    kernel.shmmni = 4096
    kernel.shmall = 4000000000
    kernel.sem = 250 512000 100 2048
    kernel.sysrq = 1
    kernel.core_uses_pid = 1
    kernel.msgmnb = 65536
    kernel.msgmax = 65536
    kernel.msgmni = 2048
    net.ipv4.tcp_syncookies = 1
    net.ipv4.ip_forward = 0
    net.ipv4.conf.default.accept_source_route = 0
    net.ipv4.tcp_tw_recycle = 1
    net.ipv4.tcp_max_syn_backlog = 4096
    net.ipv4.conf.all.arp_filter = 1
    net.ipv4.ip_local_port_range = 1025 65535
    net.core.netdev_max_backlog = 10000
    vm.overcommit_memory = 2
    ```

3. 配置/etc/security/limits.conf文件，添加以下内容(三台主机)

    ``` bash
    * soft nofile 65536
    * hard nofile 65536
    * soft nproc 131072
    * hard nproc 131072
    ```

4. 设置预读块的值为16384(三台主机 未设置)

    ``` bash
    # /sbin/blockdev --getra /dev/sda 查看预读块，默认大小为256
    # /sbin/blockdev --setra 16384 /dev/sda  设置预读块
    ```

5. 设置磁盘访问I/O调度策略(三台主机 未设置)

    ``` bash
    #echo deadline > /sys/block/sda/queue/scheduler
    ```

6. 启动ssh

    ```bash
    ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key
    ssh-keygen -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key
    ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key
    /usr/sbin/sshd
    ```

7. 创建greenplum的用户和用户组

    ``` bash
    groupadd -g 530 gpadmin
    useradd -g 530 -u 530 -m -d /home/gpadmin -s /bin/bash gpadmin
    chown -R gpadmin:gpadmin /home/gpadmin
    passwd gpadmin
    ```

8. 关闭 iptables，selinux

    ```bash
    service iptables stop
    chkconfig iptables off

    [root@mdw selinux]# cat /etc/selinux/config 
    # This file controls the state of SELinux on the system.
    # SELINUX= can take one of these three values:
    #     enforcing - SELinux security policy is enforced.
    #     permissive - SELinux prints warnings instead of enforcing.
    #     disabled - No SELinux policy is loaded.
    SELINUX=disabled
    # SELINUXTYPE= can take one of these two values:
    #     targeted - Targeted processes are protected,
    #     mls - Multi Level Security protection.
    SELINUXTYPE=targeted 

    ```

## 下载安装包并安装

1. 官网下载 https://network.pivotal.io/products/pivotal-gpdb  (这里可能需要注册)

    ``` bash
    # greenplum 安装包
    greenplum-db-5.8.0-rhel7-x86_64.zip

    # greenplum web管理界面
    greenplum-cc-web-4.1.1-LINUX-x86_64.zip

    ```

2. 上传服务器并安装

    ```bash
    unzip greenplum-db-4.2.2.4-build-1-CE-RHEL5-i386.zip

    # 这里输入安装目录(我是安装在当前用户home)
    /bin/bash greenplum-db-4.2.2.4-build-1-CE-RHEL5-i386.bin

    source ～/greenplum-db/greenplum_path.sh
    ```

3. 创建 hostlist  &nbsp; seg_hosts &nbsp; gpinitsystem_config

    ```bash
    cat conf/hostlist
    mdw
    sdw1
    sdw2

    cat conf/seg_hosts
    sdw1
    sdw2

    cat conf/gpinitsystem_config

    # Segment 的名称前缀
    SEG_PREFIX=gpseg
    # Primary Segment 起始的端口号
    PORT_BASE=33000
    # 指定 Primary Segment 的数据目录
    declare -a DATA_DIRECTORY=(/home/gpadmin/gpdata/gpdatap1  /home/gpadmin/gpdata/gpdatap2)
    # Master 所在机器的 Hostname
    MASTER_HOSTNAME=mdw
    # 指定 Master 的数据目录
    MASTER_DIRECTORY=/home/gpadmin/gpdata/gpmaster
    # Master 的端口 
    MASTER_PORT=2345
    # 指定Bash的版本
    TRUSTED_SHELL=/usr/bin/ssh
    # Mirror Segment起始的端口号
    MIRROR_PORT_BASE=43000
    # Primary Segment 主备同步的起始端口号
    REPLICATION_PORT_BASE=34000
    # Mirror Segment 主备同步的起始端口号
    MIRROR_REPLICATION_PORT_BASE=44000
    # Mirror Segment 的数据目录
    declare -a MIRROR_DATA_DIRECTORY=(/home/gpadmin/gpdata/gpdatam1 /home/gpadmin/gpdata/gpdatam2)

    ```

4. 设置环境变量，打通所有节点

    ``` bash
    # 这里需要输入 segment  gpadmin的密码 成功则 completed successfully


    gpssh-exkeys -f /home/gpadmin/conf/hostlist 

    # 批量创建文件
    gpssh -f /home/gpadmin/conf/hostlist

    mkdir gpdata

    cd gpdata

    mkdir gpmaster gpdatap1 gpdatap2 gpdatam1 gpdatam2
    ```

5. 分发安装包

    可以通过软连接的方式 更新greenplum文件位置

    ```bash

    # 打包master节点上的安装包
    tar -cf gp.tar greenplum-db/
    # 分发
    gpscp -f /home/gpadmin/conf/hostlist gp.4.3.tar =:/home/gpadmin/

    gpssh -f hostlist

    tar -xf gp.tar

    ```

6. 在每个节点上配置.bash_profile环境变量

    ```bash
    [gpadmin@mdw ~]$ cat .bash_profile 
    # .bash_profile

    # Get the aliases and functions
    if [ -f ~/.bashrc ]; then
        . ~/.bashrc
    fi

    # User specific environment and startup programs

    PATH=$PATH:$HOME/bin

    export PATH

    source /home/gpadmin/greenplum-db/greenplum_path.sh
    export MASTER_DATA_DIRECTORY=/home/gpadmin/gpdata/gpmaster/gpseg-1
    export PGPORT=2345
    export PGDATABASE=testDB

    [gpadmin@mdw ~]$ source .bash_profile

    ```

7. 初始化数据库, 默认初始化完成就启动数据库了

    ```bash
    gpinitsystem -c /home/gpadmin/conf/gpinitsystem_config -a
    ```

## Greenplum-cc-web监控软件安装

1. 运行gpperfmon_install命令

    * 创建greenplum监控用数据库(gpperfmon)

    * 创建greenplum监控用数据库角色(gpmon),后面登陆网页时使用

    * 配置greenplum数据库文件(pg_hba.conf和.pgpass)

    * 设置postgresql.conf文件，增加启用监控相关的参数。

    ```bash
    # postgresql.conf 添加
    checkpoint_segments=8
    gp_enable_gpperfmon=on
    gpperfmon_port=8888
    gp_external_enable_exec=on
    gpperfmon_log_alert_level='warning'
    gp_enable_query_metrics=on

    # 安装 gpperfmon
    gpperfmon_install  --enable  --password  gpmon  --port 2345

    # 重启
    gpstop -r

    # 查看配置是否成功
    ps -ef |grep gpmmon |grep -v grep
    ```

2. 安装 GreenplumCommand Center Console (默认所有节点都会安装)

    ```bash
    unzip greenplum-cc-web-4.1.1-LINUX-x86_64.zip

    ./greenplum-cc-web-4.1.1-LINUX-x86_64.bin

    source ~/greenplum-cc-web/gpcc_path.sh
    ```

3. 启动

    ```bash
    gpcc start
    ```

## 扩充节点

1. 按照如上配置在主机环境

2. 生成expand 配置文件

    ```bash
    cat expand

    sdw2:sdw2:33002:/data/gpdata/gpdatap1:6:2:p:34002
    sdw2:sdw2:33003:/data/gpdata/gpdatam1:7:2:m:34003

    内容包括几个字段
    hostname     主机名
    address        类似主机名
    port              segment监听端口
    fselocation   segment data目录,注意是全路径
    dbid              gp集群的唯一ID，可以到gp_segment_configuration中获得，必须顺序累加
    content                 可以到gp_segment_configuration中获得，必须顺序累加
    prefered_role        角色(p或m)(primary , mirror)
    replication_port     如果没有mirror则不需要(用于replication的端口)。
    ```

    ```bash
    # 查看现有节点情况
    select * from gp_segment_configuration ;

    # 查看节点数据目录
    select * from pg_filespace_entry ;
    ```

    配置文件可以根据以上两个表进行修改， 也可以

    ```bash
    # cat host
    sdw2

    gpexpand -f ./host

    ```

3. 运行 `gpexpand -i expand`

4. 数据重分布  `gpexpand  -d 6:00:00` (后面跟需要的时间)

5. 看着日志，错了就回滚  

参考连接 :

https://yq.aliyun.com/articles/177

https://discuss.pivotal.io/hc/en-us/articles/201202707-How-to-Use-gpexpand-Working-with-One-Host

## BUG 记录

1. Failed to complete obtain psql count Master gp_segment_configuration Script Exiti

    * 问题： 在初始化过程中，如到以下问题：

    ```bash
    gpadmin-[FATAL]:-Failed to complete obtain psql count Master gp_segment_configuration  Script Exiting!
    Script has left Greenplum Database in an incomplete state
    ```

    * 解决方法：

    ```bash
    echo "RemoveIPC=no" >> /etc/systemd/logind.conf
    /bin/systemctl restart systemd-logind.service
    ```

2. gpstart error: Do not have enough valid segments to start the array

    * 问题： gpinit 成功了在 gpstart的时候出错

    * 解决： 彻底关闭防火墙并检查是否配置开机自启

## 参考文档

[官方文档](https://gpdb.docs.pivotal.io/580/main/index.html)

[中文文档](https://gp-docs-cn.github.io/docs/common/gpdb-features.html)

[cc-web 官方文档](http://gpcc.docs.pivotal.io/410/welcome.html)