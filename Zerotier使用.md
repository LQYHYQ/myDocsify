# Zerotier使用
[toc]

# 简介
Zerotier是一款异地组网工具。每台服务器上只需要安装对应的客户端，连接到同一个网络，就可以实现 IP 互相访问。在此之上，还有自定义 DNS 服务器的功能，将通过 IP 这个步骤转换为通过域名进行访问，相当实用。

所有的设备都是客户端，连接方式是点对点。在路由器下面的话是用 uPnP 的方式进行转发实现客户端到客户端的直接连接。如果 uPnP 没有开启，会通过传统的服务器转发的方式进行连接。

## Earth
根据其介绍，将地球上的所有设备连起来。那这里的 Earth 指的就是整体的一个服务。

## Network
每一个 Network 包含的所有设备都在同一个网络里。每个网络有一个 Network ID。各客户端通过这个 ID 连接到此网络。当然，一个账号是可以创建多个网络的。

网络氛围 Public 和 Private。一般我们自己组网是要用 Private，需要在页面授权设备才可以进行访问。Public 权限好像不太有人会需要吧..

以下介绍的所有概念都是属于 Network 下的。

## Planet
指的是官方提供的服务器节点。各客户端都是通过这些服务来互相寻址的。相当于 zookeeper 的不同节点。

## Moon
自定义的 Planet。由于 Zerotier 没有国内节点，在两个设备刚开始互连的时候有可能需要通过国外的节点寻址（不过我没发现有什么慢的）导致创建连接的速度偏慢。在自己的网络里搭建 Moon 可以使连接提速。

## Leaf
客户端。就是连接到网络上的每一个设备。其实经过测试，Moon 也是客户端的一种。这里特指没有额外功能，单纯用于连接的客户端。



# 使用
Zerotier 支持基本所有设备：Windows、MacOS、iOS、Android、Linux、FreeBSD、Synology、QNAP、WD MyCloud、OpenWRT。再不济，支持 Docker，凡是能跑 Docker 能联网的设备都可以用。

## 注册账号
官方网站：https://my.zerotier.com/

可直接使用谷歌，GitHub等账号登陆。

## 创建网络
登陆进去后，直接点击 `Create A Network` 创建网络。

记住NetworkID，以后会经常用到。

## IOS
AppStore 搜索 Zerotier，下载，点击右上角+号，输入网络ID，OK。

## Linux
根据官网给的说明，直接运行脚本即可。

```Plain Text
curl -s https://install.zerotier.com | sudo bash
```
安装好后，运行命令

```Plain Text
sudo zerotier-cli join [NetworkID]
```
## 群晖
群晖建议使用Docker安装。

首先，将虚拟网卡添加到启动项：

```Plain Text
echo -e '#!/bin/sh -e \ninsmod /lib/modules/tun.ko' > /usr/local/etc/rc.d/tun.sh
chmod a+x /usr/local/etc/rc.d/tun.sh
/usr/local/etc/rc.d/tun.sh
```
以上执行后就有虚拟网卡了，然后创建 docker 容器：

```Plain Text
docker run -d           \
  --name zt             \
  --restart=always      \
  --device=/dev/net/tun \
  --net=host            \
  --cap-add=NET_ADMIN   \
  --cap-add=SYS_ADMIN   \
  -v /var/lib/zerotier-one:/var/lib/zerotier-one zerotier/zerotier-synology:latest
```
启动后，执行以下加入网络

```Plain Text
docker exec -it zt zerotier-cli join [NetworkID]
```
## 授权
每一次添加了客户端后，在控制面板上会显示新设备加入，勾上前面的勾表示对这个设备授权，会给其分配 IP，显示 zerotier 客户端的版本号和物理地址。

![image](images/QajGgzkA4_V5mIKLaQh73uoVlf9_GaZP3D2Jatx9YK8.png)



免费版最多一个网络可以有 50 个设备。

只要输入对应的虚拟 IP（Managed IPs），就可以访问对应的设备了。这些设备相当于都处于同一个局域网下了



# 高级
## 自定义 Moon
任何一个节点都可以作为 Moon 使用。当然，正常的我们肯定会选择公网 IP 的节点，一般都会使用云服务器。

官方文档：https://docs.zerotier.com/zerotier/moons

进入配置文件夹，Linux上的是：

```Plain Text
/var/lib/zerotier-one
```
执行：

```Plain Text
zerotier-idtool initmoon identity.public >>moon.json
```
打开的文件类似以下样子：

```Plain Text
{
  "id": "deadbeef00",
  "objtype": "world",
  "roots": [
    {
      "identity": "deadbeef00:0:34031483094...",
      "stableEndpoints": []
    }
  ],
  "signingKey": "b324d84cec708d1b51d5ac03e75afba501a12e2124705ec34a614bf8f9b2c800f44d9824ad3ab2e3da1ac52ecb39ac052ce3f54e58d8944b52632eb6d671d0e0",
  "signingKey_SECRET": "ffc5dd0b2baf1c9b220d1c9cb39633f9e2151cf350a6d0e67c913f8952bafaf3671d2226388e1406e7670dc645851bf7d3643da701fd4599fedb9914c3918db3",
  "updatesMustBeSignedBy": "b324d84cec708d1b51d5ac03e75afba501a12e2124705ec34a614bf8f9b2c800f44d9824ad3ab2e3da1ac52ecb39ac052ce3f54e58d8944b52632eb6d671d0e0",
  "worldType": "moon"
}
```
在其中的

```Plain Text
roots/stableEndpoints
```
数组中填上本机的 `虚拟IP/端口号` ，一般端口都是9993。如果你服务器支持 ipv6，填上 ipv6 也是不错的。

服务器要打开9993端口放行。

然后执行：

```Plain Text
zerotier-idtool genmoon moon.json
mkdir moons.d
```
会生成一个 `000xxxx.moon` 的文件，将其放在 `moons.d` 文件夹中。

这个 000xxxx 就是 moon 的ID。

在与其连接的客户端上，执行：

```Plain Text
zerotier-cli orbit 000xxx 000xxx
```
其中，第一个是 Word ID，第二个是 moon id。不用考虑细节，两个一样就对了。

这样，在连接的时候，就可以看到这台机器已经从 LEAF 变成了 MOON。

## 自定义 Controller
Controller 相当于一个完整的 Zerotier服务。



## 自定义 DNS
使用 zerotier/zeronsd 来进行 DNS 的设置。

官方文档：https://docs.zerotier.com/zeronsd/quickstart



# 待解决问题
- [ ] 同一 Network 中加入了 IPV4 设备和 IPV6 设备，无法互联互通。