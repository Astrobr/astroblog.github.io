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

### 创建属于你自己的域名

在拥有了自己的服务器以后，就可以做很多事情了。但是现在你只能通过IP地址访问自己的服务器，看起来总是有点别扭。另外，如果你想要网站有一定的影响力的话，仅有IP地址会让人几乎找不到你的网站，而且也不符合国家法律规定。所以还是建议大家弄一个自己的域名。

现在市面上的云服务器提供商也都提供域名注册的服务，直接在你的服务提供商的平台上面注册即可。下面我继续用华为云的平台演示。

首先在华为云网站页面的导航栏的搜索框内搜索“域名”，打开第一个链接“域名注册服务”。也可以直接点击这里：[域名注册服务_华为云](https://www.huaweicloud.com/product/domain.html)。

然后你可以在网页中选择你的域名，常见的如`.com`，`.cn`，`.net`等。这些域名会相对比较贵。作为学生党，我选择一个最便宜的域名`.top`，只需要9元/年。

点击你想要的域名后，会跳转到一个新的页面。接下来再次选择你要的域名，并且在“查域名”的搜索框内输入你想要的域名，看看是否已经被占用，如果被占用了就换一个。若显示“域名可注册”，就点击“立即购买”。

![域名购买](https://astrobear.top/resource/astroblog/content/buy_domain.png)

购买完成后，你就拥有了自己域名了！

### 备案

> 备案是中国大陆的一项法规，使用大陆节点服务器提供互联网信息服务的用户，需要在服务器提供商处提交备案申请。
>
> 根据工信部《互联网信息服务管理办法》(国务院292号令)和工信部令第33号《非经营性互联网信息服务备案管理办法》规定，国家对经营性互联网信息服务实行许可制度，对非经营性互联网信息服务实行备案制度。未取得许可或者未履行备案手续的，不得从事互联网信息服务，否则属违法行为。通俗来讲，要开办网站必须先办理网站备案，备案成功并获取通信管理局下发的ICP备案号后才能开通访问。[^2]

这一步不多说了，具体步骤比较繁琐，花费的时间也比较长，需要一两周。网站上有很清晰的[操作方法](https://support.huaweicloud.com/pi-icp/zh-cn_topic_0115820080.html)，请自行查阅，根据步骤操作即可。需要注意一点的是，在审核过程中可能会接到服务提供商打来的电话，不要漏接。

需要注意的是，上面的备案操作是在工信部备案的。完成了在工信部的备案以后还需要公安备案。具体[操作方法](http://www.beian.gov.cn/portal/downloadFile?token=596b0ddf-6c81-40bf-babd-65147ee8120c&id=29&token=596b0ddf-6c81-40bf-babd-65147ee8120c)也请自行查阅。

### 域名解析

在完成一系列繁琐的备案流程以后，你的网站还不可以通过域名访问。只有把你的域名跟服务器的IP地址绑定在一起之后，并且在服务器上修改了配置文件之后才可以。

首先打开管理控制台，在控制台中选择“域名注册”。然后在下面的页面中点击“解析”。

![域名注册](https://astrobear.top/resource/astroblog/content/domain.png)

点击你的域名，显示如下页面。这里显示的是你域名的记录集，前两个记录集应该是预置设置，不可暂停服务。你可以在这基础上添加自己的记录集。

![记录集](https://astrobear.top/resource/astroblog/content/record.png)

点击页面右上角红色按钮以添加记录集。添加记录集的配置如下图所示。下图中给出的例子是添加的“A”型记录集，也即通过`example.com`访问网站。若需要通过`www.example.com`访问网站，则需要为`example.com`的子域名添加“A”型记录集。具体配置参见：[配置网站解析_华为云](https://support.huaweicloud.com/qs-dns/dns_qs_0002.html#section1)。点击“确定”，完成添加。你可以通过`ping 你的域名`来测试你添加的记录集是否生效了。

![添加记录集](https://support.huaweicloud.com/qs-dns/zh-cn_image_0200891923.png)

### 配置nginx

打开你电脑上的终端，输入命令：`ssh 你的IP地址`，输入你的服务器的密码。

进入你的nginx的安装目录：`cd /usr/local/nginx/`。

使用vim打开nginx的配置文件：`vim ./conf/nginx.conf`。

按`I`开始输入。

在文件底部输入以下内容：

```nginx
server
    {
	    listen   80;                            #监听端口设为 80。
	    server_name  example.com;         #绑定您的域名。
	    index index.htm index.html;   #指定默认文件。
	    root html;           #指定网站根目录。
    }
```

然后按`esc`退出编辑，再按`Shift+zz`保存。

输入：`cd ./sbin`，切换文件夹。

执行命令：`nginx -s relod`，重启nginx服务。

这时候再尝试用浏览器访问你的域名，应该会显示之前出现过的“Welcome to nginx ”的页面了！



> 接下来将更新：给网站配置ssl证书。这也是本文的最后一次更新。



[^1]:https://support.huaweicloud.com/usermanual-vpc/zh-cn_topic_0073379079.html

[^2]: https://support.huaweicloud.com/icprb-icp/zh-cn_topic_0115815923.html