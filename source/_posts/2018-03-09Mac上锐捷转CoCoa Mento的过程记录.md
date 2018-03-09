---
title: Mac上由锐捷转CoCoaMento的过程记录
date: 2018-03-09
tags: [mac, 锐捷, 联网, 启动项]
categories: [other]
urlname: ruijie-to-cocoamento
---
***

前言：本来在Mac上用着锐捷客户端用得好好的，也不用折腾。但是前几天下载了一个`Paralles Destop`，锐捷也如约体现出它的坑爹性质——与多网卡冲突，同时也是早就受够了每次开机都要输入密码，所以综合之后决定使用`Cocoa Mento`开始了小小的折腾。

![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-03-09-130933.jpg)

<!--more-->

## 准备

-   [Cocoa Mento](http://code.google.com/p/mentohust/downloads/list)
-   startupizer 2 (用来管理开机项，后面会提及)

## 主要过程

1.  Cocoa Mento的安装及配置
    Cocoa Mento下载dmg包后把软件拖出，不要直接双击，这样会出现无法更改配置文件的错误
    
    ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-03-09-123757.jpg)

    >   网上有资料说需要更换Cocoa Mento中的8021X.exe, 我这里测试的时候是不用，如有需要，自行测试
    
    ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-03-09-123225.jpg)

1.  认证成功但无法联网问题解决
    本来经过上面的配置，把Cocoa Mento加入开机启动项即可，但是再次开机之后却发现无法联网，打开Cocoa Mento的日志发现认证已经成功了，经过多次测试，终于发现是==子网掩码错误导致无法联网==，由于是采用DHCP方式获取IP，所以无法只静态修改子网掩码（我不会。。），后来发现，点击一下`网络设置-高级`中的`DHCP续借`重新获得IP与子网掩码即可。

1.  开机运行问题
    经过上面的操作，基本的使用已经没有问题了，但是。。。。每次开机都需要这样点一下。。很烦，所以接着搞开机启动，第一时间想到了万能的`SHELL`，在网上查资料，找到在刷新DHCP的命令为`sudo ipconfig set en0 DHCP `，由于需要sudo，所以开机还要输入密码，直接写一个脚本执行命令并输入密码（密码明文，不是一个好习惯。。不管了），再用`startupizer 2`延时启动脚本，搞定。
    
    
```sh
#!/usr/bin/expect
spawn sudo ipconfig set en0 DHCP
expect "Password:"
send "你的密码\r"
interact
exec zsh
```

>   如果不加最后的`exec zsh`，我这里是会报错，不知道为什么。。


## 参考

-   [How to Renew a DHCP Lease in Mac OS X](http://osxdaily.com/2013/02/11/renew-dhcp-lease-mac-os-x/)

