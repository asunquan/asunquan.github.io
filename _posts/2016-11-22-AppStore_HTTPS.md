---
layout: post
title: "关于AppStore强制使用HTTPS"
date: 2016-11-22 15:00:00.000000000 +08:00
tags: iOS开发
---

## 前言

时光如白驹过隙, 转眼间又到了年底. 还有40多天我们就要挥别2016迎接2017了. 今年的WWDC2016开发者大会上苹果通知了我们一件事"2016年底别想消停了". 在大会上苹果宣布2017年1月1日起, 新提审的所有应用都必须启用AppTransportSecurity安全功能, 也就是说网络请求必须若原先为http协议的必须改成https协议. 由于销售业务的SDK涉及渠道众多, 在修改url的同时还要做大量的渠道SDK更新, 😔.

## HTTP和HTTPS

* HTTP: 超文本传输协议(HyperText Transfer Protocol), 是一种详细规定了浏览器和万维网服务器之间互相通信的规则, 通过因特网传送万维网文档的数据传送协议.
* HTTPS: (Hypertext Transfer Protocol over Secure Socket Layer), 是以安全为目标的HTTP通道, 简单讲是HTTP的安全版. 即HTTP下加入SSL层, HTTPS的安全基础是SSL, 因此加密的详细内容就需要SSL.  它是一个URL scheme(抽象标识符体系), 句法类同http体系. 用于安全的HTTP数据传输. https:URL表明它使用了HTTP, 但HTTPS存在不同于HTTP的默认端口及一个加密/身份验证层(在HTTP与TCP之间). 这个系统的最初研发由网景公司进行, 提供了身份验证与加密通讯方法, 现在它被广泛用于万维网上安全敏感的通讯, 例如交易支付方面.
* HTTP和HTTPS的区别
  1. HTTPS协议需要到CA申请证书, 一般免费证书很少, 需要购买.
  2. HTTP协议是超文本传输协议, 信息是明文传输; HTTPS协议则是具有安全性的SSL加密传输协议.
  3. HTTP协议和HTTPS协议使用的是完全不同的连接方式, 用的端口也不一样, 前者是80, 后者是443.
  4. HTTP协议的连接很简单, 是无状态的; HTTPS协议是由SSL+HTTP协议构建的可进行加密传输, 身份认证的网络协议, 比hHTTP协议安全.

## 解决办法

* 请公司从CA购买证书
* 请运维同学部署服务器配置SSL层证书(服务器必须支持最新的TLS1.2协议和ECDH加密算法)
* 请开发同学修改原有url为新的https链接
* 请测试同学完整测试应用所有功能, 并确保没有http请求
* 请运营同学在2017年1月1日之后应用在确保所有请求修改为https后再提交审核
* 浏览器类应用和音视频资源文件可仍然使用http链接访问

注: 服务器需要支持perfect forward secrecy(PFS)否则在请求时会失败, 若无法支持需要在Info.plist中配置对应域名不进行forwardsecrecy验证, 这样也不会被苹果审核团队拒绝.

```xml
<key>NSAppTransportSecurity</key>
     	<dict>
     		<key>NSExceptionDomains</key>
     		<dict>
                 <key>"yourdomain"</key>
                 <dict>
                     <key>NSIncludesSubdomains</key>
                     <true/>
                     <key>NSExceptionRequiresForwardSecrecy</key>
                     <false/>
                 </dict>
     		</dict>
     	</dict>
```

##### 注: 如果没有办法及时修改, 而又有上线需求, 那面也有一招, 在2017年到来之前提审并留出时间供苹果审核, 之后发布. 不过这个终究不是长远之计, 1月1日之后再提审必须做好充分的修改.