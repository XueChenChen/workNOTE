
# 导读
1. [React-native 基础教程](## React-native 基础教程)
1. [lottery 相关](## lottery 相关)
1. [React-native apk打包](## React-native apk打包)
1. [协议规则:HTTPS](## 协议规则:HTTPS)
1. [IPV6](## IPV6)

[TOC]

## React-native 基础教程
- [React Navite 中文网](http://reactnative.cn/docs/0.47/getting-started.html)

## lottery 相关
- 前端： RNLotteryProject/APP
  - android: `index.android.js`->`main.js`
  - iOS: `index.ios.js` -> `main.js`
- 后台：我提供一个postman分享集
  - 链接 [陆合彩postman地址](https://www.getpostman.com/collections/abea09c4a7015af2d25e)

## React-native apk打包
 - [打包教程http://blog.csdn.net/u010411264/article/details/54237446](http://blog.csdn.net/u010411264/article/details/54237446)

## 协议规则:HTTPS
> 2017年1月1日起，苹果App Store中的所有App都必须启用ATS(App Transport Security)安全功能。

**启用ATS后，它会屏蔽明文HTTP资源加载，强制App通过HTTPS连接网络服务，不满足以下条件，ATS都会拒绝连接。**
### 1. ATS客户端支持：
  - https走域名方式请求的，无需更改。
  - https走IP直连方式，使用私有证书走442端口的，将不再可行；需要端口改成443端口，客户端预埋证书改为CA证书，并且需要设置域名白名单。
  - 测试的时候，将ATS的开关打开，Allow Arbitrary Loads设为NO，并且保证手机系统是iOS9以上。
  ![](assert\v2-df101faa852d3822952131af8d64b375_b.jpg)

### 2. 服务端支持要求：
  - 服务器必须支持传输层安全(TLS)协议1.2以上版本;
  - 证书必须使用SHA256或更高的哈希算法签名;
  - 必须使用2048位以上RSA密钥或256位以上ECC算法等

| 被允许的加密方式      |    苹果ATS检测 |
| :-------- | --------:|
| [Cocoa Keys](https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html)  | [苹果ATS检测](https://www.qcloud.com/product/ssl#userDefined10) |


## IPV6
  - windows: [配置方法](http://jingyan.baidu.com/article/22fe7ced67c9443002617f94.html)
  - mac: [配置方法](http://www.cnblogs.com/fengmin/p/5526487.html)
> 测试 1. [乌龟是否动了](http://www.kame.net/kame-mosaic.html)

  ![dianji](assert\20170805161620.png)
  ![kandonglema](assert\20170805161648.png)
