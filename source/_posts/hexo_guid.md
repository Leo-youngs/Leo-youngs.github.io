---
title: ubuntu 安裝配置 hexo
---


## 环境介紹
主机： ubuntu 18.04

### 安裝依賴

``` bash
sudo apt-get install nodejs
sudo apt install npm
sudo apt install git
npm install -g hexo
```


### hexo 創建

``` bash
mkdir hexo_init
cd hexo_init
hexo init
hexo g
hexo s
```
> 登錄 http://localhost:4000  查看效果

### 下載主題
``` bash
git clone https://github.com/tufu9441/maupassant-hexo.git themes/maupassant
npm install hexo-renderer-pug --save
npm install hexo-renderer-sass --save
```

### 修改yml文件
```
theme: maupassant

deploy: 
  type: git
  repository: https://github.com/Leo-youngs/Leo-youngs.github.io
  branch: master
  message: update
```



### 生成靜態文件

``` bash
$ hexo generate
```

更多信息: [Generating](https://hexo.io/docs/generating.html)

### 發布

``` bash
$ hexo deploy
```

更多信息: [Deployment](https://hexo.io/docs/deployment.html)
