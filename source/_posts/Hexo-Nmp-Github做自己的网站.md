---
title: Hexo+Npm+Github搭建自己的网站
date: 2017-02-18 19:28:41
reward: true
categories: "Hexo教程"
tags:
	- Hexo
	- 技术
---
### 前言  

&emsp;&emsp;经过三天断断续续的折腾，个人定制版Blog终于上线！成就感还是相当不错滴！由于网上的教程层次不齐，遂将本次的安装调试过程记录如下。本文分5大块进行详细介绍：  
1. hexo + git 软件的配置安装与使用
2. 自定义个性化域名
3. 网页主题的选取
4. 评论区的添加，站长统计分析工具的部署以及网站浏览量的设置
5. 相册的添加与维护   

<!--more-->

### part1：软件的配置
> 声明：本次操作系统环境为Ubuntu16.04，windows下的配置其实差不多，只是安装软件方式的差别。

1. **软件安装 git，nodejs和npm**   
``` bash
sudo apt-get install git  
sudo apt-get install npm  
sudo apt-get install nodejs  
sudo apt-get install nodejs-legacy  
#更新nodejs  
npm update -g  
npm install -g n  
n latest
```

2. **注册github账户并创建Repository**  
&emsp;&emsp;去github官网www.github.com注册账号,注册完成之后点击界面右上角+号，New repository，创建新的代码托管仓库。*注意：仓库名必须为：yourname.github.io,其中yourname为你注册的账户名字*     
![](http://i1.piimg.com/567571/e2856a7a63dca8c1.png)
3. **正式安装Hexo**  
在用户目录建立一个文件夹，作为博客根目录，比如直接在用户目录创建一个blog文件夹。  
``` bash
mkdir blog  
cd blog  
sudo npm install -g hexo  
hexo init
hexo generator  
hexo server
```
这个时候可以看到blog文件夹下会生成一个public文件夹，里面有一些编译之后生成的html，js等脚本。此时在浏览器中输入http://localhost:4000 应该就可以看到网站的首页了。  

4. **部署到远程托管网站上**  
刚才的网页展示还只是本地使用，现在要将其部署到github网站上利用网址进行访问。  
``` bash
设置github信息
git config --global user.name "cnfeat"//用户名
git config --global user.email  "cnfeat@gmail.com"//邮箱
```
找到blog目录下的_config.yml,对其进行一些配置修改，必须修改的地方为最后。其中repo改为自己的网址
``` python
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo:https://github.com/leopardpan/leopardpan.github.io.git
  branch: master
```  
修改完成之后，继续在命令行中输入：  
``` bash
npm install hexo-deployer-git --save  
hexo d  //进行远程部署
```  
在这个过程中会要求你输入github网站的用户名和密码进行登录认证。当然也可以通过设置SSH Key的方式进行自动认证，这里不再详述。这个时候去github网站，可以看到自己刚才新建的代码仓库已经存放的有数据了，而且就是public文件夹的数据。  
现在，在浏览器中输入yourname.github.io 即可访问自己的网页了。

---  
以上就是简易安装的全部教程了，以后如果再想写新的文章，只需要：  
``` bash
hexo new "newname"
hexo g  //生成
hexo d  //部署
```   
其他常见命令请自己查询。

---  
### part2：自定义个性化域名  
&emsp;&emsp;由part1可以知道，想要访问自己的网站，必须输入yourname.github.io才行，那么如何按照自己想要的方式进行呢？这就需要自己进行域名注册了。所幸腾讯对于我们这种穷学生有照顾，推出1元云主机加域名活动，大家可以自行申请。  
域名注册完成之后，需要按照如下步骤完成绑定工作：   
1. **设置github**
找到自己的github仓库，点击右上角setting按钮，找到Custom domain选项进行设置，输入自己的域名并保存即可。  
![](http://i1.piimg.com/567571/438f634b71aaf9aa.png)   

![](http://p1.bqimg.com/567571/6d344aca7b57c285.png)  

2. 到域名注册中心进行解析  
&emsp;&emsp;登录腾讯云，管理中心-云产品-域名服务-云解析。会显示你自己的注册域名，点击解析-添加记录，记录类型选择CNAME，主机记录为www，记录值为你的待解析网址：yourname.github.io  
![](http://i1.piimg.com/567571/b5b006033b421ac3.png)   
待解析完成，就可以通过你的自定义域名进行访问了。腾讯云的解析不是很快，而且需要实名认证。还是阿里云比较好。  

---  
### part3：网页主题的选取  
&emsp;&emsp;Hexo自带的网站主题不太美观，可以自己选择比较不错的主题来进行替换。各个版本的主题都在github上有开源。   

*今天写的太累了，以后再接着写。。。*
