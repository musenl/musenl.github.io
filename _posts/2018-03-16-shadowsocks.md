layout: post
title: 基于Bandwagon的VPS服务使用ShadowSocks+cow代理
categories: shadowsocks
description: 基于Bandwagon的VPS服务使用ShadowSocks+cow代理
keywords: ShadowSocks, Bandwagon,vps

## VPS哪家强

不要问度娘“VPS哪家强”，它只是一只广告狗。于是，我去微博搜了“VPS”关键字，发现了更真实的用户体验，有人被Bandwagon经济实惠的VPS感动到哭了！

选中了Bandwagon一年9.99刀后，用PayPal（可绑银联卡）华丽丽地支付掉，邮箱立马收到了几封邮件，瞬间到手IP和SSH Port，世界变化太快了～

![1.png](http://7vzozf.com1.z0.glb.clouddn.com/2015/02/2137450562.png)

------

## 用ShadowSocks科学上网

之前一直使用Pennyjob科学上网，碰到人多也没法。现在有了自己的VPS，OpenVPN Server和Shadowsocks Server等都可以任性地搭建了。

个人偏爱[ShadowSocks](https://shadowsocks.com/)，轻巧强大。

> ShadowSocks服务器端提供了各种版本，如Python、Nodejs、Go、C libev等等，安装配置过程极其简单。而用户端则可以在windows、mac、iOS和android上轻松运行，很好很强大。

> shadowsocks实质上也是一种socks5代理服务，类似于ssh代理。与vpn的全局代理不同，shadowsocks仅针对浏览器代理，不能代理应用软件，比如youtube、twitter客户端软件。

打开KiviVM Control Panel，好贴心的一键安装啊，瞬间就装好Shadowsocks Server。

![2.png](http://7vzozf.com1.z0.glb.clouddn.com/2015/02/1121814647.png)

当然，安装服务器端也可以使用python，具体可以参考 https://pypi.python.org/pypi/shadowsocks

根据IP、Port和Password配置好客户端的Shadowsocks的config.json文件：

```
{
"server":"my_server_ip",
"server_port":8388,
"local_port":1080,
"password":"barfoo!",
"timeout":600,
"method":"table" //加密方案推荐 aes-256-cfb ，需安装 m2crypto
}
```

启用全局代理，打开浏览器就可以飞了。在[ping.pe](http://ping.pe/)这可以测试自己的IP。 另外也可以改用PAC模式，需要先从GFWList更新PAC。这样就只是局部地对某些被墙网站进行代理。

开启自动代理模式：需要下载浏览器额外使用插件（如Chorme的Proxy SwitchyOmega）：

增加全局代理模式设置：配置代理端口

![img](http://images.cnitblog.com/blog2015/658832/201504/041241585291276.png)

 

配置自动切换模式，Proxy SwitchyOmega的代理服务器模式即是全局模式；自动切换模式和PAC模式都是局部模式，其中自动切换模式在设置完切换规则后，可以设置规则列表网站（如https://autoproxy-gfwlist.googlecode.com/svn/trunk/gfwlist.txt），并自动同步。

![img](http://images.cnitblog.com/blog2015/658832/201504/041243377178472.png)

------

## cow代理

COW 是一个简化穿墙的 HTTP 代理服务器。它能自动检测被墙网站，仅对这些网站使用二级代理。

项目官网地址：<https://github.com/cyfdecyf/cow>  (有附中文使用说明)

然后配置 cow 。官方文档： [cow github](https://github.com/cyfdecyf/cow#cow-climb-over-the-wall-proxy)以 windows 版本为例 。下载：[cow](http://dl.chenyufei.info/cow/) 。也可以从 go 源码编译。windows 版本的 cow 配置文件为目录下 rc.txt 。实际情况下，我们需要的部分很少。

 

 ![img](http://images.cnitblog.com/blog2015/658832/201504/041311095763788.png)

COW 的设计目标是自动化，理想情况下用户无需关心哪些网站无法访问，可直连网站也不会因为使用二级代理而降低访问速度。

作为 HTTP 代理，可提供给移动设备使用；若部署在国内服务器上，可作为 APN 代理 支持 HTTP, SOCKS5, shadowsocks 和 cow 自身作为二级代理 可使用多个二级代理，支持简单的负载均衡 自动检测网站是否被墙，仅对被墙网站使用二级代理 自动生成包含直连网站的 PAC，访问这些网站时可绕过 COW。

## PS：

- Shadowsocks各版本语言的Server略有不同，但是都可以支持多端口（libev最小最快，但是多端口支持需要改造）。
- Proxy SwitchyOmega的代理服务器模式即是全局模式；自动切换模式和PAC模式都是局部模式，其中自动切换模式在设置完切换规则后，可以设置规则列表网站（如https://autoproxy-gfwlist.googlecode.com/svn/trunk/gfwlist.txt），并自动同步。
- Shadowsocks中设置的代理端口和SwitchyOmega中代理服务器的代理端口要一致。

标签: [bandwagon](http://www.stackess.com/index.php/tag/bandwagon/), [vps](http://www.stackess.com/index.php/tag/vps/), [shadowsocks](http://www.stackess.com/index.php/tag/shadowsocks/)

 