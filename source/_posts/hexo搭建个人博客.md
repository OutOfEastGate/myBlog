---
title: hexo搭建个人博客
date: 2022-03-20 10:54:07
tags: 教程
category: 教程
---

---
# 安装Nodejs
[安装](https://www.runoob.com/nodejs/nodejs-install-setup.html)

# 查看node版本
node -v	
# 查看npm版本
npm -v	
# 安装淘宝的cnpm 管理器
npm install -g cnpm --registry=http://registry.npm.taobao.org	
# 查看cnpm版本
cnpm -v	
# 安装hexo框架
cnpm install -g hexo-cli
# 查看hexo版本    
hexo -v
# 创建blog目录	
mkdir blog
# 进入blog目录	
cd blog	 
# 生成博客 初始化博客
sudo hexo init
# 启动本地博客服务 	
hexo s	
# 本地访问地址
http://localhost:4000/	
# 创建新的文章
hexo n "我的第一篇文章"  
# 清理
hexo clean 
# 生成
hexo g 
# Github创建一个新的仓库 YourGithubName.github.io
# 在blog目录下安装git部署插件o
cnpm install --save hexo-deployer-git 
# 配置_config.yml 
Deployment
Docs: https://hexo.io/docs/deployment.html

deploy:

type: git

repo: https://github.com/YourGithubName/YourGithubName.github.io.git

branch: master

# 部署到Github仓库里
hexo d
# 访问这个地址可以查看博客
https://YourGithubName.github.io/  

 # 下载yilia主题到本地
 git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia 

# 修改hexo根目录下的 _config.yml 文件
 theme: yilia
# 清理一下
hexo c	
# 生成
hexo g	
# 部署到远程Github仓库
hexo d	
# 查看博客
https://YourGithubName.github.io/  