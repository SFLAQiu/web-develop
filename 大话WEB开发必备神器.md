---
layout: post
title:  "大话WEB开发必备神器"
date:   2016-10-18 12:00:05 +0800
categories: 开发 神器
tags: 开发神器  
author: SFLYQ
---

* content
{:toc} 


开发的过程中经常会使用到的各种辅助软件，学会并灵活的使用这些工具，可以提高开发效率，提高排查问题的速度，达到一个事半功倍的效果；  

![image](http://blog.thankbabe.com/imgs/sq.jpg)   







这里我就列出在开发的过程中我会使用的一些工具，分享给大家。




---

### 抓包神器

> WEB API 开发和调试，线上问题排查，总是需要有抓包工具进行请求的抓包分析     
> 如：手机APP，PC 软件，浏览器和WEB API 交互请求的抓包 
 

* 常用功能
    * 模拟请求，get，post 等
    * 获取请求报文，响应报文，分析报文
    * 编辑请求报文(请求参数，请求头，cookie 等)，重新发起请求
    * 请求断点，编辑请求头，或者响应内容 等
    * 并发请求测试
    * 请求响应时间分析
    * ... ...

> 以下是我经常会使用到的几个抓包神器,具体的使用看【使用教程】， 我这里就分享下我在使用工具上的心得

#### Fiddler

![image](http://blog.thankbabe.com/imgs/fiddler.png)   
[使用教程](http://www.cnblogs.com/TankXiao/archive/2012/02/06/2337728.html)

> Fiddler 是我最早开始使用的抓包器，完全免费，把报文的解析成不同的数据模块通过切换可以很清晰的找到自己想要数据，
而且可以很方便的修改请求头数据，上传的参数，cookie，编辑完成后还可以重新发起请求；

* 优点：
    * 完全免费
    * 支持手机端代理抓包
    * 支持https抓包
    * http 报文解析成对应的数据模块，结构清晰查找方便
    
* 缺点
    * Fiddler 是基于微软 .Net 技术开发的，没办法直接在 Mac/Linux 下使用 
    * host解析修改后需要重新启动，不然解析无法生效
    
---  

#### Charles 

![image](http://blog.thankbabe.com/imgs/Charles.png)   
[使用教程](http://blog.csdn.net/lmmilove/article/details/50244537)


* 优点：
    * Fiddler的优点 Charles基本都有
    * 支持Mac/Linux
    * Charles抓包到的请求根据域名分类展示，这点我很喜欢，很方便查看
    * 可以发起并发请求

* 缺点
    * 收费，免费的有限制

---

#### Wireshark

![image](http://blog.thankbabe.com/imgs/Wireshark.png)   
[使用教程](http://www.cnblogs.com/TankXiao/archive/2012/10/10/2711777.html)

> 这个抓包工具功能比较强大，除了可以抓 http协议还可以抓如：TCP 等 各种协议的请求，
我一般在抓程序发起TCP连接操作Redis，mongodb的时候会使用，看看代码在实际的协议中触发的操作命令是什么。

---

### HOST解析工具

#### SwitchHosts
![image](http://blog.thankbabe.com/imgs/shost.png) 

[官方入口](https://oldj.github.io/SwitchHosts/)

> 域名host解析必备神器，支持 windows和Mac的开源工具，
使用简单，支持自定义分类配置，可切换配置，合并配置。  

---

### 轻量级文本编辑器

#### Sublime

![image](http://blog.thankbabe.com/imgs/sublime.png)

> 轻量级文本编辑器，各种使用技巧，前端开发神器，各种插件欲罢不能，如：Emmet 等

---

#### Visual Studio Code

![image](http://blog.thankbabe.com/imgs/vscode.png)

> 微软开源轻量级文本编辑器后起之秀，据说打开大文本文件速度很快，也是支持各种插件，官方提供中文版，
对于：nodejs，typescript 等开发也都有很好的支持，有一种要取代sublime的“感脚”

---

### 翻墙神器

#### Lantern
![image](http://blog.thankbabe.com/imgs/lantern.png)

> Lantern 中文简称“蓝灯”，免费翻墙工具，翻墙后就可以通过google查询开发资料对于开发者来说这个还是很有必要的，同时也是宅男必备神器╮(‵▽′)╭
此处省略一万字，自行脑补画面  

---

### 浏览器调试工具

>打开浏览器F12就会启动调试工具，不同内核浏览器集成不一样的开发者工具

* `[推荐]`谷歌开发者调试工具
    * 可切换手机模式
    * 设置网络状况,2G，3G，4G，wifi 等
    * 请求抓包
    * html查看，编辑
    * css查看，编辑
    * cookie，本地数据等查看，编辑
    * js断点debug
    * 控制台警告，错误提示，调试js
    * 性能分析
    * 谷歌开发者工具是英文版，刚开始使用会比较不习惯，用多了就习惯了
    * 等
    
    
* 火狐的Firebug
    * 与谷歌开发者工具基本功能差不多
    * 火狐Firebug是中文容易上手

* IE开发者调试工具
    * 基本功能大同小异

---

### 必备浏览器插件

* JSONView
* Postman 支持发各种请求发送，支持自定义类目保存请求云同步，自定义脚本

---

> 发布时间：2016-10-18


