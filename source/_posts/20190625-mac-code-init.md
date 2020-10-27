---
title: mac coding
date: 2019-06-25 21:50:51
tags: 日常
categories: 后端
---

## Mac 环境下配置开发环境

记录Mac日常coding中部分软件的安装
主要是记录经常使用以及需要破解的软件

## 安装brew

[brew 中文官网](https://brew.sh/index_zh-cn)

```bash

/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

```

## 安装python

``` bash
brew install python3

python3 -V
```

## 安装Java

``` bash
# 使用增强的 homebrew
brew tap caskroom/versions

brew cask install java8

# 卸载
brew cask uninstall java8

```

## 安装 Navicat Premium Mac

### 安装

1. 进入[官网](http://www.navicat.com.cn/products )点击免费使用
2. 找到对应的Mac 版本点击下载

### 激活

1. 下载DoubleLabyrinth/navicat-keygen [github地址](https://github.com/DoubleLabyrinth/navicat-keygen) release -> mac

    ```bash
    git clone -b mac https://github.com/DoubleLabyrinth/navicat-keygen.git


    cd navicat-keygen

    make all

    ```

2. 生成激活码

    ```bash
    cd bin

    ./navicat-patcher /Applications/Navicat\ Premium.app/Contents/MacOS/Navicat\ Premium


    ## 打开钥匙串新建 navicat 并总是信任

    codesign -f -s "foobar" /Applications/Navicat\ Premium.app/

    ## 选择版本并输入相关信息
    ./navicat-keygen ./RegPrivateKey.pem

    ## 打开navicat 点击注册 填入上一步注册码

    ## 断网选择离线激活，复制请求码，继续key-gen ,回车 回车

    ### 不出意外获得激活码

    ```

3. 激活成功

## 安装IntelliJ IDEA

1. 下载[官网](https://www.jetbrains.com/idea/download/#section=mac)

2. 激活  -> [激活文档](http://idea.lanyus.com/)
