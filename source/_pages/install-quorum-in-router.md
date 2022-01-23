---
title: 在家用Linux软路由OpenWrt上部署quorum的平淡历程
date: 2021-01-23 11:31:00
tags: ["RUM", "quorum", "OpenWrt", "Linux"]
---

## 我的思考

前面的文章中提到，我已经可以编译quorum并且在VPS上稳定运行，自己的电脑和手机也运行着RUM-APP。我在想，将来RUM是要遍地开花的，而且需要足够数量的public节点形成规模。还有哪些设备可以让quorum跑起来？于是我想起了软路由。

## 去年购买的软路由

去年，为了整理家里的网络架构，并且把我家和父母老家的网络内部局域网打通，然后内部共享文件、照片、电影等，我买了台软路由。通过OpenVPN，两地和VPS建立了一个简单的VPN网络。之后因为太稳定了，我都忘记了这台软路由的存在。今天打开管理界面看，已经300多天没有重启了。

![image-20220123094841229](C:\Users\samso\AppData\Roaming\Typora\typora-user-images\image-20220123094841229.png)

我的路由器是6网口的。去年在某宝购买的时候，请店家帮忙安装好OpenWrt x86-64版本，收件后经过简单配置就可以使用。

![image-20220123085629272](C:\Users\samso\AppData\Roaming\Typora\typora-user-images\image-20220123085629272.png)

![image-20220123085244210](C:\Users\samso\AppData\Roaming\Typora\typora-user-images\image-20220123085244210.png)

## 开始折腾
从OpenWrt的网页管理界面就可以打开终端，输入路由器的管理账号和密码登录。

![image-20220122153100060](C:\Users\samso\AppData\Roaming\Typora\typora-user-images\image-20220122153100060.png)

然后，执行 `mkdir /data/quorum` 创建quorum的运行目录。并上传预先编译好的quorum程序。

![image-20220123083433162](C:\Users\samso\AppData\Roaming\Typora\typora-user-images\image-20220123083433162.png)

![image-20220122153522777](C:\Users\samso\AppData\Roaming\Typora\typora-user-images\image-20220122153522777.png)

上传成功后，执行以下几个命令，把quorum移动到运行目录，并授予执行的权限。

```bash
mv /tmp/upload/quorum /data/quorum/
cd /data/quorum
chmod u+x quorum
```

![image-20220123083742378](C:\Users\samso\AppData\Roaming\Typora\typora-user-images\image-20220123083742378.png)

![image-20220123083842017](C:\Users\samso\AppData\Roaming\Typora\typora-user-images\image-20220123083842017.png)

我忘记在防火墙开放端口4722了，赶紧补上配置，这样可以保证quorum运行后是public节点。

![image-20220123112437532](C:\Users\samso\AppData\Roaming\Typora\typora-user-images\image-20220123112437532.png)

![image-20220123112542940](C:\Users\samso\AppData\Roaming\Typora\typora-user-images\image-20220123112542940.png)

![image-20220123112728965](C:\Users\samso\AppData\Roaming\Typora\typora-user-images\image-20220123112728965.png)

接着就可以尝试运行quorum，我这里执行如下命令并置于后台运行。其中openwrt-quorum-peer是Keystore密码，-ips参数是路由器的本地IP和VPN IP地址。

```bash
RUM_KSPASSWD=openwrt-quorum-peer /data/quorum/quorum -peername homeowrt -ips 10.253.253.1,10.0.0.6 -listen /ip4/0.0.0.0/tcp/4722 -apilisten :4723 -peer /ip4/94.23.17.189/tcp/10666/p2p/16Uiu2HAmGTcDnhj3KVQUwVx8SGLyKBXQwfAxNayJdEwfsnUYKK4u,/ip4/132.145.109.63/tcp/10666/p2p/16Uiu2HAmTovb8kAJiYK8saskzz7cRQhb45NRK5AsbtdmYsLfD3RM &
```

![image-20220123084039402](C:\Users\samso\AppData\Roaming\Typora\typora-user-images\image-20220123084039402.png)

成功执行后，我把上面的quorum启动命令放到路由器的启动项脚本中，这样每次重启路由器都可以自动运行。

![image-20220123110855557](C:\Users\samso\AppData\Roaming\Typora\typora-user-images\image-20220123110855557.png)

最后，执行以下命令，把JWT Token和证书内容输出，复制转贴到RUM桌面版的外部节点配置中。

```bash
cat /data/quorum/certs/server.crt
curl -k -X POST -H "Content-Type: application/json" https://127.0.0.1:4723/app/api/v1/token/apply
```

![image-20220123084431876](C:\Users\samso\AppData\Roaming\Typora\typora-user-images\image-20220123084431876.png)

## 最终运行效果

![image-20220123084252296](C:\Users\samso\AppData\Roaming\Typora\typora-user-images\image-20220123084252296.png)

## 后记

折腾过程还是蛮顺利的，得益于基于golang语言编写的quorum的跨平台特性。OpenWrt的使用和配置也很方便，整个过程1小时左右，关键点在于程序的上传、防火墙规则的设置以及quorum参数的配置。大家如果身边有OpenWrt软路由，也可以尝试搞起来，遇到问题可以到RUM中文群（7000100109）一起讨论。感谢阅读！
