---
title: greenplum 折腾 
date: 2018-05-08 10:52:58
tags: 数据仓库  大数据 greenplum
---

### Greenplum是一个MPP（海量并行处理）计算框架的分布式数据库，其数据库引擎层是基于著名的Postgresql数据库，企业级数据库产品，现已开源。Greenplum拥有丰富的特性，包括：
1. 完全支持ANSI SQL 2008标准和SQL OLAP 2003 扩展，支持ODBC和JDBC
2. 支持分布式事务，支持ACID
3. 支持行存储、列存储，以及可通过外部表的方式访问其它关系型数据库或者Hadoop
4. 拥有良好的线性扩展能力，支持上千个节点
 

## 详情参考  
[ **docker中安装配置Greenplum集群**](https://my.oschina.net/u/876354/blog/1606419)



### BUG 记录
> Failed to complete obtain psql count Master gp_segment_configuration Script Exiti

> 问题： 在初始化过程中，如到以下问题：
```
gpadmin-[FATAL]:-Failed to complete obtain psql count Master gp_segment_configuration  Script Exiting!
Script has left Greenplum Database in an incomplete state
```
 

> 解决方法：
```bash
echo "RemoveIPC=no" >> /etc/systemd/logind.conf
/bin/systemctl restart systemd-logind.service
```

## 多节点docker集群 init 成功 gpstart 不成功
> 问题 gpinit 成功 在gpstart的时候 超时（gp_segment_connect_timeout ），应该是在与segment通信的时候连接失败


## 宿主机安装 greenplum 集群
> 1.  **gpinit 成功了在 gpstart的时候出错**  ` gpstart error: Do not have enough valid segments to start the array` 
> 2. 关闭防火墙 centos74 关闭防火墙 发现连网失败 初始化的报错 ` FATAL:  DTM initialization: failure during startup recovery, retry failed, check segment status (cdbtm.c:1513)` 好在服务启动成功（实际上还是失败，安慰）
> 3. 再次打开防火墙 暴力打开 配置文件内配置的端口 20000以上的端口 （telnet 测试端口是后开放）
> 4. gpstart 启动成功 ` Database successfully started` (激动)
> 5. 装完以为完事，在插入查询的时候莫名的卡死，查询文档后得知 主从节点在通讯的时候是需要1024:65535 udp 通讯， 再次iptables 打开端口


## 详细的文档 [ Greenplum数据库文档](https://gp-docs-cn.github.io/docs/best_practices/intro.html)