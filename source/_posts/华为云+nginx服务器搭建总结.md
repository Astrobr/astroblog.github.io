---
title: 华为云+nginx服务器搭建总结
date: 2020-1-8 10:29
categories: 
	- [CS]
	#- [cate2]
	#...
tags: 
	- Nginx
	- Internet server
	- Network Technology
	- Experience
	#...

#If you need a thumbnail photo for your post, delete the well number below and finish the directory.
thumbnail: https://kinsta.com/wp-content/uploads/2018/03/what-is-nginx.png

#If you need to customize your excerpt, delete the well number below and input something. You can also input <!-- more --> in your article to divide the excerpt and other contents.
excerpt: 搭建自己的服务器并不难，只是过程较为复杂。

#You can begin to input your article below now.
---

> 由于自己是去年七月配置好的服务器，有一些细节或者遇到的问题已经记不太清，故本文可能会有不完整的地方，遇到问题请善用搜索引擎，而且服务器的配置方法也不只有这一种。本文主要用作对自己操作步骤和方法的一个总结，以便于日后查阅。本文章将持续更新。

### 购买服务器

首先去[华为云官网](https://www.huaweicloud.com/?locale=zh-cn)注册一个账号。如果是学生，可以搜索“学生”，并进行学生认证。学生认证的步骤参见[学生认证流程](https://support.huaweicloud.com/usermanual-account/zh-cn_topic_0069253575.html)。进行身份验证后可以购买学生优惠套餐，云服务器价格只要99元/年，比阿里云和腾讯云的都要便宜一些。

![华为云学生优惠](https://astrobear.top/resource/astroblog/content/hwcloud_discount.png)

购买完成后，你可以在控制台看到自己现有的资源以及运行情况。

![控制台](https://astrobear.top/resource/astroblog/content/console.png)

### 配置安全组

> 安全组是一个逻辑上的分组，为具有相同安全保护需求并相互信任的云服务器提供访问策略。安全组创建后，用户可以在安全组中定义各种访问规则，当云服务器加入该安全组后，即受到这些访问规则的保护。
>
> 系统会为每个用户默认创建一个默认安全组，默认安全组的规则是在出方向上的数据报文全部放行，入方向访问受限，安全组内的云服务器无需添加规则即可互相访问。默认安全组可以直接使用。
>
> 安全组创建后，你可以在安全组中设置出方向、入方向规则，这些规则会对安全组内部的云服务器出入方向网络流量进行访问控制，当云服务器加入该安全组后，即受到这些访问规则的保护。[^1]

在控制台点击“弹性云服务器ECS”，在这里你可看到你的服务器的公网IP，请记下这个IP地址。然后点击在列表中点击你的服务器的名称。

![选择服务器](https://astrobear.top/resource/astroblog/content/security_groups.png)

进入云服务器管理页面后，点击“安全组”。再点击“Sys-default”可以看到默认安全组。然后下面给出的图片是我目前的安全组设置，仅供参考。选择“入/出方向方向规则”，再点击“添加规则“即可手动添加规则。一般来说，配置的都是入方向的安全组，并且源地址（访问服务器的设备的IP地址）都为“0.0.0.0/0”（所有IP地址）。

通常需要配置如下几个功能：

- SSH远程连接Linux弹性云服务器（协议：SSH，端口：22）
- 公网“ping”ECS弹性云服务器（协议：ICMP，端口：全部）
- 弹性云服务器作Web服务器
  - 协议：http，端口：80
  - 协议：https，端口：433

详细配置请参考[安全组配置示例](https://support.huaweicloud.com/usermanual-ecs/zh-cn_topic_0140323152.html)。

![安全组设置](https://astrobear.top/resource/astroblog/content/sg_settings.png)

![安全组设置](https://astrobear.top/resource/astroblog/content/sg_settings1.png)

配置完成后，可以打开电脑上的终端，用下面的语句测试一下：

`ping 你的公网IP`

出现类似下面的内容就代表成功了：

![ping测试](https://astrobear.top/resource/astroblog/content/ping_test.png)

你可以按下`Ctrl+C`来结束`ping`这个进程。

然后在终端里输入：

`ssh 你的公网IP`

如果你的安全组配置正确的话，会让你输入服务器的登录密码。输入密码（注意：密码是不会显示的）后回车，应该可以看到这样的输出：

![ssh登录](https://astrobear.top/resource/astroblog/content/ssh_login.png)

这个时候，你的终端就已经连接上了服务器的系统了，你在终端里的一切操作都是作用在服务器上的。

### 在服务器上安装nginx

首先请在终端使用ssh登录你的服务器，然后按照下面给出的顺序输入命令。

```shell
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel #安装编译工具及库文件
cd /usr/local/ #切换到目标安装文件夹
wget http://nginx.org/download/nginx-1.16.1.tar.gz #下载最新版本的Nginx
tar -zxvf nginx-1.16.1.tar.gz #解压文件
cd nginx-1.16.1 #进入解压的文件夹
./configure #执行程序
make #编译
make install #安装
cd /usr/local/nginx/sbin #进入Nginx安装目录
./nginx #运行Nginx
```

此时，安装应该已经完成了。打开浏览器，在地址栏中输入你的公网ip。如果看到下图所示内容，就代表安装成功了。

![nginx安装成功](https://astrobear.top/resource/astroblog/content/nginx_install.png)

> 接下来将更新的内容：购买域名及绑定...

[^1]:https://support.huaweicloud.com/usermanual-vpc/zh-cn_topic_0073379079.html







