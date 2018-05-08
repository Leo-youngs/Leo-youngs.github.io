---
title: Linux 使用记录
date: 2018-05-08 10:07:55
tags: echo EOF
---

# linux 命令终端显示-bash-4.2#解决方法
> 近期在折腾docker centos 使用别人的镜像 发现终端显示 `bash-4.2#` 很是忧伤（之前遇到，好记性不如烂笔头）

``` bash
# 这里的root 可以替换成你的用户
cp /etc/skel/.bashrc /root/  
cp /etc/skel/.bash_profile  /root/  

```
> 然后 exit 重新登录就好了


# linux 文件编辑
> 在线上操作文件 一般都是 vi 但是 vi 适用于各种花式操作 这里介绍实用的简单的操作
 

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

EOF是END Of File的缩写,表示自定义终止符.既然自定义,那么EOF就不是固定的,可以随意设置别名,在linux按ctrl-d就代表EOF, **EOF一般会配合cat能够多行文本输出.**