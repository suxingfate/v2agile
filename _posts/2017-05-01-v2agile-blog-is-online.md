---
layout:     post
title:      "v2agile.com is alive"
subtitle:   ""
date:       2017-05-01 01:00:00
author:     "Vincent"
header-img: "img/post-bg-2015.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 折腾
---

# 个人博客上线了 v2agile.com

>上学的时候写过个人博客，但是当时穷学生，舍不得花钱买空间和域名，一直没有弄起来。而且作为软件从业者，自己启动个tomcat，写点jsp建个网站觉的做出来很难看，主要是前端不好看。最近因为经常需要记录一些文字，而且也喜欢Markdown的文档，碰巧有个同事自己有建立自己的站点，请教了之后于是结合自己的一些喜好，搭建了这个站点。

## 站点名字
http://www.v2agile.com
[http://www.v2agile.com](http://www.v2agile.com)

## 站点结构

### 服务器
采用的是板瓦工的廉价服务器

- **http://banwagong.cn/ 优惠信息网站**
- **http://bandwagonhost.com/ 官网**

项目     | 指标
-------- | ---
内存 | 512M
硬盘    | 10G
流量 | 1000G
机房 | LOS ANGELES
价格 | 12$/6 Months


-------------------

### 域名服务


- **https://www.godaddy.com/**

－网上有人提到有汇率差，比如用卢布可以节省30％的价格，但是我测试之后并没有，反而发现只有人民币本币的价格公道。

－另外一个是可以用优惠码。不过我注册之后系统貌似提供了一个默认优惠码，优惠幅度远高于搜索到的其他优惠码 `GENNBACN29` 成交价格是116RMB 两年。
 
 －DNS配置
 在godaddy的页面里面选择manage DNS，然后输入需要管理的域名，然后修改A纪录就可以了，默认CNAME WWW也是指向@，也就是第一项A纪录的设置IP。填入我的服务器IP即可，几乎立刻生效了。

### Software Stack

####  wordpress

- **https://lnmp.org/**

wordpress的安装参考上面的链接即可，非常简单
这样就可以完成wordpress的安装，然后可以愉快的写博客了。

然后我又很想写md的文档，觉得wordpress的功能不够灵活。于是走上了另外一条路

#### jekyll

之前玩过GitPages，在上面放过一个jekyll做的站点。觉得不够漂亮，于是找了下模版，发现了一个神器

- **http://huangxuan.me/**

这是一个前端大神的作品，幸好是开源的，于是拿过来改一下就可以上线了。

#### 百度统计 

- **http://tongji.baidu.com/**

因为上面的静态页面没有统计功能，于是可以使用百度提供的统计接口进行统计，同时也是很好的站点存活数据观察的地方。

方法很简单，注册之后会提供一段js代码。把代码放到`_includes/head.html` 的js区域，然后重新生成全部代码. 并把代码上传到服务的服务目录就可以了。

```
jekyll serve —watch
rsync -avz  -e 'ssh -p xxxx' _site root@23.106.150.18:/tmp/
```

#### Google Ad

一直想挂点广告玩玩，于是也同时申请了Google Ad的账号，不过还在审批中

---------

[1]: http://www.v2agile.com
[1]: http://www.v2agile.com/index.html


