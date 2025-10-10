---
title: "小程序抓包"
description: "利用抓包工具对微信小程序抓包"
pubDate: "2025-9-10"
image: https://cdn2.zzzmh.cn/wallpaper/origin/dd4aee16880411ebb6edd017c2d2eca2.jpg/fhd?auth_key=1762272000-2e3e48e8a038fad74ae8ccf2ecc244d59cb6539e-0-5b3beff0611a9523824c25118da4b9df
categories:
  - 实践
tags:
  - Linux
badge: 实践
---



# 小程序抓包

## 前言

本文主要用Burpsuite对微信小程序进行抓包，但此过程不仅限于对微信小程序抓包，也可以抓取其他应用的数据包。由于Burpsuite抓包需要配置代理，但代理可能会影响微信的正常应用，所以采用Charles或Proxifier将数据包转发给Burpsuite来实现Burpsuite抓取数据包的过程。此方法不仅能抓取PC端的数据包，也能抓取PE端的数据包。



## 一、准备工作

### 1.下载安装

[Burpsuite下载地址](https://portswigger.net/burp/communitydownload)

https://portswigger.net/burp/communitydownload

[Charles下载地址](https://www.charlesproxy.com/)

https://www.charlesproxy.com/

[Proxifier下载地址](https://www.proxifier.com/download/)

https://www.proxifier.com/download/



### 2.安装证书

#### 2.1 Charles

![安装证书1.png](https://s2.loli.net/2025/10/10/rHCo84a3NO75LpR.png)

打开顶部Help -> SSL Proxying -> Install Charles Root Certificate, 安装证书要放在受信任的根证书颁发机构目录下。



#### 2.2Burpsuite

浏览器打开链接https://burp，点击下载安装证书，安装证书要放在受信任的根证书颁发机构目录下。



## 二、抓取数据

### 1.Charles&Burpsuite

首先打开BurpSuite的代理设置，确保填写完整代理服务器和端口，HTTP/S数据包只要流经这个地址就会被Burpsuite捕捉到。

![Burp代理设置](https://s2.loli.net/2025/10/10/9GgQkwMeP5JE8hZ.png)

然后打开Charles代理设置，因为本文抓取的是PC端微信小程序数据包，所以要打开Windows Proxy选项。

![Charles设置1](https://s2.loli.net/2025/10/10/TgxAruiSWhBpZCe.png)



之后打开Proxy Settings设置Charles的代理端口，并勾选Enable transparent HTTP proxying和Support HTTP/2选项。

![Charles设置2](https://s2.loli.net/2025/10/10/tAKVTop2XanuPyF.png)

Enable transparent HTTP proxying 表示 Proxifier 能够**自动拦截系统中所有 HTTP 请求**，无需应用显式配置代理。启用后即使程序未手动设置 HTTP 代理（如浏览器、命令行工具等），Proxifier 也能自动劫持并转发它们；这让你无需每个软件单独设置代理，特别适用于“全局代理”模式；但有极少数程序（尤其是使用非标准网络栈或 HTTPS 直接连接的）可能出现兼容问题。

Support HTTP/2表示代理支持 **HTTP/2 协议**。启用它后Proxifier 会尝试与支持 HTTP/2 的目标网站或程序使用 HTTP/2 协议通信；若代理或目标服务器不支持 HTTP/2，会自动回退到 HTTP/1.1。



接着再打开 SSL Proxying Settings 选项，勾选 Enable SSL Proxying ，在Include中添加地址，Host和Port中填入*。

![Charles4.png](https://s2.loli.net/2025/10/10/c2rb3j1kYnESdFp.png)

Charles 默认只能抓 HTTP，不抓 HTTPS。启用 Enable SSL Proxying Charles 就会作为“中间人”（MITM）代理：

拦截 HTTPS 请求

用 Charles 自签的证书（Charles Root Certificate）假装成目标网站

浏览器信任这个证书后，Charles 就能解密并查看 HTTPS 请求内容

在 “SSL Proxying Settings” 里你可以配置哪些域名要被解密。默认 Charles 不会自动对所有网站解密，以防性能或安全问题。Host和Port中填入*表示对所有域名、所有端口的 HTTPS 流量都启用解密。



然后打开External Proxy Settings，勾选Use external proxy servers，并且勾选左侧栏的 Web Proxy 和 Secure Web Proxy ，并在 Web Proxy Server 处填写ip地址和端口号（注意ip地址和端口号要和前面Burpsuite的代理设置相同，并且左侧栏的勾选的两个选项都要填写）。

![Charles3.png](https://s2.loli.net/2025/10/10/s1TmUMbHJRGlhfv.png)

左侧栏勾选的两项分别表示转发HTTP流量和HTTPS流量。

左侧的第三个选项 “SOCKS Proxy”（SOCKS 代理）代表一种底层的通用代理协议，它和前两个（HTTP / HTTPS 代理）不同，不是专门为网页设计的，而是可以转发任意 TCP 或 UDP 流量。这里我们用不到，所以不用勾选。



这时我们打开微信小程序就可以在Burpsuite的HTTP历史记录里面发现抓到的数据包。并且我们正常用Burpsuite进行数据包的拦截修改和重放。

![1.png](https://s2.loli.net/2025/10/10/2gtZ4P7CaKNoLpz.png)





### 2.Proxifier&Brupsuite

Burpsuite设置同上。

打开Proxifier，Profile -> Advanced 勾选Infinite Connection Loop Detection 和 Infinite Name Resolution Loop Detection。

![Proxifier1.png](https://s2.loli.net/2025/10/10/w5g2CQ7TvVFtqN3.png)

Infinite Connection Loop Detection（无限连接循环检测），防止代理设置错误时，流量在系统与代理间无限循环（例如代理自身又被代理）。启用后 Proxifier 能检测并阻止这种情况，避免 CPU 飙高或死循环。

Infinite Name Resolution Loop Detection（无限域名解析循环检测）防止 DNS 请求被不断重定向或反复解析（例如本机代理和系统代理互相转发）。



之后打开HTTP Proxy Servers 勾选 Enable HTTP proxy servers support，让Proxifier获取到的数据能转发到Burpsuite中。

![Proxifier2.png](https://s2.loli.net/2025/10/10/sAF6QLZe9BK5Ucd.png)



然后打开Proxy Servers 设置要转发到的服务器地址。添加ip地址和端口号（同Burpsuite设置）。Protocol选择HTTPS。

![Proxifier3.png](https://s2.loli.net/2025/10/10/Df59XBqs1icx2ja.png)



再打开 Proxification Rules 设置代理规则。添加规则，由于我们要抓取微信小程序数据包，所以我们在Applications中填写wechat*.exe，表示抓取前缀wechat进程是数据包。下面的Action选取刚才设置的服务器地址。注意Proxifier规则的代理权重从上到下依次降低，所以我们新建的规则要放在第一行。

![Proxifier4.png](https://s2.loli.net/2025/10/10/TsKO5PxazS1vhrV.png)

之后就能在Burpsuite上美美抓包了。
