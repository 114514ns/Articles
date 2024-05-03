---
title: HZResearch
published: 2024-05-03
description: ''
image: ''
tags: ['逆向','校园网']
category: '逆向'
draft: false 
---




# HZ Research

## 目录

[TOC]

## Intro

本文初稿写于2023年7月中旬，经过多次修改。

现在是2024年5月。

虽然我认为我写的还算详细，但阅读本文仍需具备一定的相关知识。



高三的上机课还是挺多的，绝对有必要好好利用，从5月份进入高三到现在，已有了些成果，正好现在放暑假了，就想着把这些研究成果记录下来，就有了这篇文章。



## 上网

通过观察，可以发现负责“上网认证”的设备是深信服的“行为管理AC”

![image-20230716224009865](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230716224009865.png)

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgqq_pic_merged_1689518462536.jpg" alt="qq_pic_merged_1689518462536" style="zoom: 25%;" />

闲鱼上两三百就能搞到一台。

在他们的论坛上可以找到比较详细的资料

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230716224525959.png" alt="image-20230716224525959" style="zoom: 50%;" />

可以发现这玩意好像还挺牛逼的

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230717160036193.png" alt="image-20230717160036193" style="zoom:50%;" />

但是不影响咱们玩游戏或者上网，下面就介绍几种上网的方法及其原理。

### DNS隧道

如果你在没有登录的情况下，随便ping一个域名，应该会看到这样的提示：

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230717155656445.png" alt="image-20230717155656445" style="zoom:33%;" />

从输出的信息中可以看到，你ping的是一个域名，然后它把域名解析成了ip。

有没有发现什么问题？解析域名，肯定是要和外部网络的DNS服务器通信的；你没有登录，但是这样的通信仍然可以进行。

说明在没有登录的情况下仍然可以使用DNS协议。

这当然也是有原因的，因为如果不能正常解析域名，当打开一个网站的时候，就直接报错，这样就不能实现自动重定向到登录页面了。

所以可以利用这点，搭建一个代理服务器，主机和代理服务器之间的通信伪装成DNS协议。

但是怎么伪装呢？经过测试，**只要某一个协议是基于UDP的，并且使用53端口，就可以免登录直接用，不会被阻断，本文介绍的方法都是基于这个原理**

#### v2ray

知道了上面这些结论，那么常规方法，当然是使用v2ray。

v2ray是一个用来翻墙的软件，在这里也可以用作绕过上网认证。

> **V2Ray**，是Victoria Raymond以及其社区团队开发的**Project V**下的一个工具。Project V是一个工具集合，号称可以帮助其使用者打造专属的基础通信网络。Project V的核心工具称为V2Ray，其主要负责网络协议和功能的实现，与其它Project V通信。V2Ray可以单独运行，也可以和其它工具配合，以提供简便的操作流程。开发过程主要使用[Go语言](https://zh.wikipedia.org/wiki/Go语言)，Core采用[MIT许可证](https://zh.wikipedia.org/wiki/MIT许可证)并[开放源代码](https://zh.wikipedia.org/wiki/开放源代码)。
>
> 在[中国大陆](https://zh.wikipedia.org/wiki/中国大陆)，本工具广泛用于突破[防火长城](https://zh.wikipedia.org/wiki/防火长城)（GFW），以访问被封锁和屏蔽的内容。
>
> 2019年2月，V2Ray项目创始人Victoria Raymond突然消失，其Twitter、Telegram以及知乎停止更新。
>
> 2019年8月2日，原作者Victoria Raymond的Telegram 频道提示：“创建此频道的用户的帐户在过去5个月中处于非活动状态。如果它在接下来的30天内仍然不活动，那么该账户将自动销毁，并且这个频道将不再拥有创建者。”[[7\]](https://zh.wikipedia.org/zh-cn/V2Ray#cite_note-7)
>
> 原作者的Github账号依然保持更新直到2019年11月最后一次提交commits。

![image-20230909193731814](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230909193731814.png)

原作者在2019年11月 在欧洲消失。此后v2ray改名v2fly，由社区接手继续开发。

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230909204537859.png" alt="image-20230909204537859" style="zoom:50%;" />

##### 搭建

既然是代理服务器，那肯定是需要一台vps的，如果需要翻墙的话需要选择国外的服务器。

但是国内的服务器需要实名认证，而且价格贵带宽小，建议直接选择国外服务器，比如这个

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230909193824461.png" alt="image-20230909193824461" style="zoom:50%;" />

一年190RMB，一个月500G，500Mbps的带宽，性价比很高。

买好了之后随便安装一个Linux系统，这里我选的是Ubuntu，连上ssh，安装x-ui（x-ui是一个简单易用的管理x-ray节点的面板）

```bash
bash <(curl -Ls https://raw.githubusercontent.com/vaxilu/x-ui/master/install.sh)
```

根据提示安装完之后打开面板

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230909194144899.png" alt="image-20230909194144899" style="zoom:50%;" />

注意端口要选择53，协议选择KCP（KCP是UDP+TCP缝合出来的，底层还是UDP）

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230717160512990.png" alt="image-20230717160512990" style="zoom:33%;" />

这样v2ray节点就搭建完成了

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230909194518653.png" alt="image-20230909194518653" style="zoom:33%;" />



如果没有服务器或者不想自己搭建的话，也可以去买现成的。

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230717160715815.png" alt="image-20230717160715815" style="zoom:50%;" />

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230717160737013.png" alt="image-20230717160737013" style="zoom:33%;" />

1个月11块，1588g，可以用于绕过校园网，当然也可以翻墙，还可以用于手机流量的免流（这个自行研究）

##### 运行

**注意所有基于v2ray的客户端只提供系统代理，这样只有浏览器和其他遵循系统代理的软件才能上网，如果有其他软件上网的需要，需要使用sstap，通过虚拟网卡的方式强制让所有软件走系统代理**

v2ray原生的客户端是CLI程序，也就是没有GUI界面，所以可以选择一个v2ray的gui封装

QV2ray 基于QT C++编写 不需要任何依赖，支持32位 已经停更，但是不影响使用 注意需要手动下载内核

首先下载qv2ray https://github.com/Qv2ray/Qv2ray/releases

和v2ray https://github.com/v2fly/v2ray-core/releases 注意v2ray-core的版本不要高于5

然后点击首选项 设置内核路径

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230930122933238.png" alt="image-20230930122933238" style="zoom:33%;" />

把绕过中国大陆这个选项勾掉，否则国内网站不会走代理

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230930123000281.png" alt="image-20230930123000281" style="zoom:33%;" />

如果需要在局域网内共享代理，把监听ip改成0.0.0.0

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230930123027728.png" alt="image-20230930123027728" style="zoom:33%;" />

然后让别人设置好ip和端口

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230930123100209.png" alt="image-20230930123100209" style="zoom:50%;" />

就可以上网冲浪了。



几个注意点

- v2ray的vmess协议会校验系统时间，误差不能超过90秒，否则无法正常使用。机房系统的时间是有问题的，建议在搭建节点时候选择vless协议，因为vless不需要验证时间。
- 不要使用clash，因为clash只支持vmess上的ws协议，不支持kcp。

#### Java

如果你所在的环境无法使用u盘，那么就不能直接使用上面的方法了。

学校的32bit系统里，有个Dreamweaver，它的安装目录下有JDK1.6，可以自己做一个简单的文件传输协议来获取v2ray

Server.java

```java
    public static void main(String[] args) {
        try (DatagramSocket clientSocket = new DatagramSocket()) {
            InetAddress serverAddress = InetAddress.getByName(SERVER_IP);

            BufferedReader userInput = new BufferedReader(new InputStreamReader(System.in));
            System.out.print("请输入要请求的文件名：");
            String filename = userInput.readLine();

            byte[] sendData = filename.getBytes();
            DatagramPacket sendPacket = new DatagramPacket(sendData, sendData.length, serverAddress, SERVER_PORT);
            clientSocket.send(sendPacket);

            byte[] receiveData = new byte[BUFFER_SIZE];
            DatagramPacket receivePacket = new DatagramPacket(receiveData, receiveData.length);
            clientSocket.receive(receivePacket);

            System.out.println("收到服务器的响应：" + receivePacket.getData().length);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

Client.java

```java
package org.example;

import org.apache.commons.io.FileUtils;

import java.io.*;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.net.URL;

public class Server {
    private static final int PORT = 12345;
    private static final int BUFFER_SIZE = 1024;

    public static void main(String[] args) {
        try (DatagramSocket serverSocket = new DatagramSocket(PORT)) {
            System.out.println("服务器已启动，等待客户端连接...");

            byte[] receiveData = new byte[BUFFER_SIZE];
            byte[] sendData = new byte[BUFFER_SIZE];

            while (true) {
                DatagramPacket receivePacket = new DatagramPacket(receiveData, receiveData.length);
                serverSocket.receive(receivePacket);

                InetAddress clientAddress = receivePacket.getAddress();
                int clientPort = receivePacket.getPort();

                String request = new String(receivePacket.getData(), 0, receivePacket.getLength());
                System.out.println("收到来自客户端的请求：" + request);

                sendData = getFileContent(request);

                DatagramPacket sendPacket = new DatagramPacket(sendData, sendData.length, clientAddress, clientPort);
                serverSocket.send(sendPacket);

                System.out.println("已发送响应到客户端：" + sendData.length);

                receiveData = new byte[BUFFER_SIZE];
                sendData = new byte[BUFFER_SIZE];
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static byte[] getFileContent(String filename) {
        File file = new File(filename);
        FileInputStream fileInputStream = null;
        try {
            fileInputStream = FileUtils.openInputStream(file);
            return fileInputStream.readAllBytes();
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        } finally {
            assert fileInputStream != null;
            try {
                fileInputStream.close();
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }
    }
}
```

上面只是半成品，UDP单个包最大只支持64k，所以需要拆分。

完全有比这个更简单的方法，但这是我第一个想到的方法，所以就在这里提一下。

#### HTTP/3

HTTP1.1和HTTP/2是基于TCP的

> **HTTP/3**是第三个主要版本的[HTTP](https://zh.wikipedia.org/wiki/HTTP)协议。与其前任[HTTP/1.1](https://zh.wikipedia.org/wiki/超文本传输协议#HTTP/1.1)和[HTTP/2](https://zh.wikipedia.org/wiki/HTTP/2)不同，在HTTP/3中，将弃用[TCP](https://zh.wikipedia.org/wiki/传输控制协议)协议，改为使用基于[UDP](https://zh.wikipedia.org/wiki/用户数据报协议)协议的[QUIC](https://zh.wikipedia.org/wiki/快速UDP网络连接)协议实现。[[1\]](https://zh.wikipedia.org/wiki/HTTP/3#cite_note-1)
>
> 此变化主要为了解决HTTP/2中存在的[队头阻塞](https://zh.wikipedia.org/wiki/队头阻塞)问题。由于HTTP/2在单个TCP连接上使用了[多路复用](https://zh.wikipedia.org/wiki/多路复用)，受到TCP[拥塞控制](https://zh.wikipedia.org/wiki/拥塞控制)的影响，少量的丢包就可能导致整个TCP连接上的所有流被阻塞。
>
> QUIC（快速UDP网络连接）是一种实验性的[网络传输协议](https://zh.wikipedia.org/wiki/网络传输协议)，由[Google](https://zh.wikipedia.org/wiki/Google)开发，该协议旨在使网页传输更快。在2018年10月28日的邮件列表讨论中，[互联网工程任务组](https://zh.wikipedia.org/wiki/互联网工程任务组)（IETF） HTTP和QUIC工作组主席[Mark Nottingham](https://zh.wikipedia.org/w/index.php?title=Mark_Nottingham&action=edit&redlink=1)提出了将HTTP-over-QUIC更名为HTTP/3的正式请求，以“明确地将其标识为HTTP语义的另一个绑定……使人们理解它与QUIC的不同”，并在最终确定并发布草案后，将QUIC工作组继承到HTTP工作组。[[2\]](https://zh.wikipedia.org/wiki/HTTP/3#cite_note-2) 在随后的几天讨论中，[Mark Nottingham](https://zh.wikipedia.org/w/index.php?title=Mark_Nottingham&action=edit&redlink=1)的提议得到了IETF成员的接受，他们在2018年11月给出了官方批准，认可HTTP-over-QUIC成为HTTP/3。[[3\]](https://zh.wikipedia.org/wiki/HTTP/3#cite_note-3)
>
> 2019年9月，HTTP/3支持已添加到[Cloudflare](https://zh.wikipedia.org/wiki/Cloudflare)和[Google Chrome](https://zh.wikipedia.org/wiki/Google_Chrome)（Canary build）。Firefox Nightly在2019年秋季之后添加支持。[[4\]](https://zh.wikipedia.org/wiki/HTTP/3#cite_note-4)
>
> 2022年6月6日，IETF正式标准化HTTP/3为RFC9114[[5\]](https://zh.wikipedia.org/wiki/HTTP/3#cite_note-5)。

UDP协议，有没有想到什么？

把服务端的端口改成53，就可以实现***免登录且不需要任何工具***地访问这个网站，你可以在这个端口上搭建一个网盘，然后就可以下载到v2ray客户端。

还有更高级的玩法，用siteproxy

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230909204831599.png" alt="image-20230909204831599" style="zoom:50%;" />

这东西可以直接在网页上代理，连v2ray也不用了。

这个我还没研究明白，这段内容下回补上。

### 窃取密码

~~上面介绍的方法，可能会遇到各种问题，比如服务器被墙，或者v2ray炸了之类的，如果能拿到密码，就比较保险了。~~

23.9.9更新 ： 老师可能不会直接在电脑上登录，我暂时也没发现其他登录的方式，以下方法的实际的可行性有待考证，但是原理是正确的。



#### ARP攻击

ARP，即地址解析协议，是一种在OSI数据链路层中使用的协议。它的作用是将IP地址解析为MAC地址，以供交换机使用（因为交换机是根据MAC地址来转发数据的）。

具体来说，交换机会向整个局域网发送ARP包。当某台设备收到这个包时，会判断包中的IP地址是否与自己的IP地址相符。如果是，该设备将响应自己的MAC地址。

这样，交换机就能知道一个IP地址对应的MAC地址，并将其保存在自己的缓存中。

然而，以太网是一种广播式网络，一个主机发送的数据包其他主机也可以看到。因此，攻击者有可能通过应答ARP包，在短时间内大量回应，以覆盖交换机缓存中正确的MAC地址。接着，被欺骗的主机所发送的数据将不会被正确地发送到目标主机，而是被发送到攻击者的主机上。攻击者可以查看和修改这些数据，这就是ARP欺骗。

如果攻击者只是欺骗交换机，那么受害主机的流量，发送到攻击主机上，就没有后续了，这样受害主机就断网了

如果欺骗交换机后再把流量正确地转发，那么就可以看到被害主机的流量，而且被害主机不会觉察到任何异常

参考：[图解ARP协议（二）ARP攻击篇](https://zhuanlan.zhihu.com/p/28818627)

具体到操作，可以在windows下使用arpspoof+wireshark

arpspoof的github https://github.com/alandau/arpspoof

wireshark的官网 https://www.wireshark.org/

整个过程 简单地说，就是用arpspoof实现让目标主机的所有流量经过自己的主机，然后把流量正确地发送到目的地然后返回给目标主机。

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230801122030609.png" alt="image-20230801122030609" style="zoom:33%;" />

然后用wireshark查看经过自己主机的流量。

arpspoof已经开始工作了。然后打开wireshark

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230801122127613.png" alt="image-20230801122127613" style="zoom:33%;" />

可以看到，wireshark里捕获到了我手机的http流量。

这样的攻击 ，防御的方法也是有的，可以在交换机里把ip和mac的对应关系写死，但是咱学校应该是不会的。

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230716224800902.png" alt="image-20230716224800902" style="zoom: 33%;" />

#### 代理服务器抓包

这种方法和arp攻击类似，但不是欺骗交换机，而是主动控制老头的电脑，然后设置代理服务器，然后就可以看到他所有的http流量，就能拿到密码了。

也是有风险，而且也比较麻烦，不整了。

## 硬盘保护卡

学校的电脑是Acer的品牌机，但是在网上没有找到相同配置的，搜到的硬盘保护卡的资料也很少。

但是没有关系，能搞到密码就能随便玩了。

我能弄到密码也是机缘巧合，我通过注册表获得的极域的管理密码，然后突然想到 有没有可能硬盘保护卡的密码和极域的一样。

结果就是一样的。



2024.1.12更新

获取密码的方法：在任务管理器找到EzMonitor.exe进程，右键点击创建转储文件，然后找到dump文件的位置，用记事本打开，搜索win7，在搜索到的附近就可以看到明文的密码。不需要借助任何工具。

很显然，硬盘保护卡管理器的设计是有问题的，它在软件启动时，从硬件获取明文密码，读取到内存里，然后在用户输入的时候，把用户输入的字符串和之前获取的密码对比。

这样的设计，对一般人来说，没什么问题；但是稍微懂点技术，就可以轻松拿下。

### U盘

搞到了密码，登录硬盘保护卡，把禁止usb设备这个选项勾掉就好了

### 硬盘

这块硬盘保护卡把分区分为两种类型，一是系统分区，不支持直接修改，如果要修改得创建进度（创建完进度还可以回退到以前的进度，类似git）

二是数据分区，可以修改，默认会在每次重启后还原内容。

如果需要保留自己的数据，进入分区管理，把数据分区的还原方式设置为**"每30天清除"**即可，其实过了30天也不会清除的。

### 引导

如果想从自己的u盘启动系统，正常情况是是去bios里把启动盘设置成自己的u盘。

但是，在这里就不行了，硬盘保护卡会使用自己的引导，无视bios的设置。

这就不太好办了，我暂时想到的方法是使用Clover，就是添加一个Clover的系统，再用Clover引导自己的u盘，相当于套两层引导。

或者在硬盘里安装Windows，再在机房的Windows7手动添加启动项，这需要修改C盘。

暂时没有尝试。



2024.1.12更新

如果不能从硬盘引导，那么**“机房管理员”**是如何安装系统的？所以一定有办法可以从u盘引导，因为不然就装不了系统了。

通过查阅说明书得知，在系统引导界面，按下**Home**键然后输入密码，就可以进入管理员模式，在这里可以选择要引导的系统所使用的进度。

光标选择一个空白的系统，按下**Ctrl+O** 然后选择自己的u盘，然后就可以启动自己的系统了

比如你可以在一块移动硬盘上安装一个win10，然后启动，这样不会对原来的系统产生任何影响。



## 远程控制

### VNC

VNC是一个远程控制软件，在学校的Windows7 32位系统有这个软件，而且没有设置密码。

这就意味着 你可以只要知道别人的ip，就可以随便控，包括老师，学校的三个机房网络是通的，所以控制其他班的人也是可以的。

左边的机房 老师的ip是192.168.44.13	 右边的是192.168.44.101

下载VNC-Viewer<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230909205735834.png" alt="image-20230909205735834" style="zoom:50%;" />

输入ip就可以直接控制了。

另外vnc似乎支持udp协议，把端口改成53就可以免登录控制家里的电脑

自行研究吧。

### Remote Desktop

这是Windows系统自带的远程控制，但是默认是不开启的，需要手动开启<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230909210137854.png" alt="image-20230909210137854" style="zoom:33%;" />

而且系统账号必须有密码。设置好然后连接就可以了。<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230909210327136.png" alt="image-20230909210327136" style="zoom:33%;" />



可能稍有些麻烦，但是它的性能绝对是吊打其他所有同类软件，有着近乎原生的体验，支持直接拽托复制文件。

其他软件的原理是在被控端收集显卡渲染好的画面，然后压缩 发给主控端。

而Remote Desktop是把画面传到客户端渲染，减少了网络流量，性能也提高了。




## 每日交作业

虽然这个和上网无关，但考虑到可能有些人需要，就把这部分内容加进来吧。

### 原理

其实没啥高级的。按f12打开开发者工具，然后再打开作业详情界面

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230930130052444.png" alt="image-20230930130052444" style="zoom:33%;" />

可以看到，浏览器向服务器发送了getWorkDetail请求，看看内容<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230930130217374.png" alt="image-20230930130217374" style="zoom: 50%;" />

可以猜得出来，submitCover就是作业的答案。

所以可以得出结论，不管作业是否设置为老师可见，服务器返回的信息都会包含别人的答案，只是不会显示出来而已。

以此类推，其他的api可能也有类似的设计缺陷，或者说bug。

大部分api都没有验证用户的权限，可以自己开个老师的账号，然后抓包拿到api，啥都能看

你甚至可以修改，删掉老师布置的作业<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20230930130620721.png" alt="image-20230930130620721" style="zoom:33%;" />



以前还有个更严重的漏洞，可以随意获取作业，不管是不是自己班的，然而，返回的作业内容里包含用户信息，只要遍历所有的作业，就可以获取平台内所有用户信息，信息包括 用户名字，手机号，学校，班级和同学。

当然，现在这个漏洞修复了

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231002205354441.png" alt="image-20231002205354441" style="zoom:33%;" />

是我提交的。没有拿到一分钱。其实，这种体量的用户信息，在telegram上可以卖到几千USDT



### 第三方客户端

其实用开发者工具已经可以看到别人答案了，但是这样太麻烦了。

所以我做了一个QQ Bot，可以直接在QQ内抄答案，我的qq号封了，暂时没法演示。

源码：https://github.com/114514ns/WorkBot



在去2022年10月的时候，当时是疫情放假在家，我就写了个web端，这是我学react后做的第一个项目，界面有些简陋，但是能用。

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231002212145264.png" alt="image-20231002212145264" style="zoom: 25%;" />

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231002212221506.png" alt="image-20231002212221506" style="zoom:25%;" />



一年后的现在，国庆假期，花了两天时间重新写了一遍，不只是可以看作业，甚至可以删作业改作业批作业，还有更好看的布局和ui，

这应该算是我一年的进步了

github：https://github.com/114514ns/dailyworkweb

预览：https://work.ikuntech.xyz/login

<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231002212414105.png" alt="image-20231002212414105" style="zoom:25%;" />

![image-20240503203221070](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/image-20240503203221070.png)

![image-20240503203242553](C:\Users\zhang\AppData\Roaming\Typora\typora-user-images\image-20240503203242553.png)

## 离线内容

如果机房里没有网，或者你不会使用代理，那也没关系。只要你的u盘里有足够丰富的内容，网络其实不是很重要。

### Wikipedia

应该不会不知道维基百科吧？可以在网上下载到离线阅读器和离线数据库。数据库有3G的精简版和22G的完整版，可以自己选择。维基百科的内容还是非常多的，建议下载一个。

### PVZ

经典的小游戏。我在可以插u盘后第一个想到的游戏就是PVZ。在此后半年的时间里，大多数人的娱乐，就是PVZ。

### Minecraft

学校的电脑不是很好，但是玩个Minecraft还是没有太大压力。
![re](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/2024-03-14_08.31.54.png)
图中的版本是1.12.2 with Optifine，多人游戏。
大多数情况下都能有150+FPS，还是可以流畅游玩的。
建议1.12原版加上高清修复就够了，不要玩高版本或者mod，因为32位的JVM只能分配1G的内存，1.12原版，刚刚好。

#### 多人游戏

有没有想过班上十几个人联机玩mc的感觉？
可以在64位的系统上部署一台Spigot服务器，这样还能分担客户端的压力
首先，需要去网上获取spigot服务端，可以自己编译，也可以去minecraftversion.net上获取。
然后

``` bash
java -jar spigot-1.12.2.jar
```

然后第一次启动会在服务端目录下生成elua.txt文件，
需要把文件内的false改为true，表示同意MinecraftELUA
然后再次修改server.properties
改为

```text
online=mode:false ##关闭正版验证
```

再次启动服务器，就可以愉快地多人联机啦。
![](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/2024-03-14_09.40.59.png)
![](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/2024-03-14_09.09.35.png)

服务器里人最多的时候有11个，几乎每个人都考了290分以上。


## 考试系统

不知道有没有发现这个程序？
![](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/cfg.JPG)
注意第一个按钮，检测数据库连接。
为什么客户端要检测与数据库的连接？有两种可能

- 让服务端检测它与本地数据库的连接
- 客户端直接连接数据库，检测连接

抓个包看一下
![](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/wireshark.JPG)
看到了TDS协议的数据包，说明客户端直接与SQL Server进行数据交互。

既然这样，那我们就要进数据库一探究竟了...
wireshark没有显示登陆凭据，而客户端配置工具也没有填写帐号密码的地方，那么，我们可以猜测它的密码是在程序里写死的。

这个程序使用了UPX加壳，解压一下

```bash
upx -d C_CONFIG.EXE
```

然后打开火绒剑和配置工具，然后选择提取文件字符串，往下翻几下，可以发现账号和密码，账号：wlks
密码;WWJTwlks

![](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/dump.jpg)

拿到了账号密码，就可以用office套件内的access连接上数据库
![](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/access-1.JPG)
点击**外部数据**选项卡下的**导入并链接ODBC数据库**

导入数据库有两个选项

- 导入  把数据库的内容导入access，本地的更改不会同步到服务器
- 链接  本地的修改会同步到服务器上

建议先选第一个选项熟悉一下，然后再选第二个

然后需要添加数据源
![](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/addSrc.jpg)
选择最后一个，SQL Server
![](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/addSrc2.jpg)

名字可以随便写一个

服务器这一栏，会自动发现局域网内的SQLServer服务器，填写完之后点下一步
![](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/addSrc3.jpg)

账号和密码即为上面拿到的账号密码

账号 wlks

密码 WWJTwlks

然后再次点击**导入并链接数据库**，输入密码
就可以看到服务器上的所有表了
![](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/access-3.JPG)

可以看到大量拼音命名的表和字段

比较重要的表有

- DAZI 打字题库
- OfficeSuCai Office的素材和答案
- KaoSheng 存储着考生信息和状态 如果这个账号正在考试，那么还会有他的答案和成绩
- ChengXuGaiCuo 改错题库
- ChengXuSheJi 编程题库 有答案，但答案不是明文，可能是经过某种编码或加密，我还没研究出来
- KaoShiCanShu 存储考试参数，包括考试时间，考试标题，监考密码等

![](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/access-5.JPG)
这些内容都可以读取和更改，所以。。。
![](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/login.JPG)

![](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/type.JPG)

也可以拿到Word Excel DreamWeaver 的答案。

和别人的考试答案，改分数，想干啥就干啥

使用Access导出的数据库，二进制的数据在xml里是以base64存储的

这是我写的Parser，需要输入Access导出的xml文件

效果:
![](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/answer1.JPG)

![](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/answer2.JPG)

服务器及数据库文件：[链接](https://1drv.ms/f/s!AkiLwgcEeGbDyCUbwlWAbM3MWi79?e=DKP8jg)

## 硬件

### U盘

u盘肯定是必备的，
买质量好点的，免得丢数据。

### 网卡

如果机房没有网络，那么要上网就需要一张无线网卡，然后手机开热点，风险较大，也不是很有必要，不建议。

### 蓝牙

如果需要在机房看视频，听歌，就需要USB蓝牙来连接耳机，如果有需要可以整一个。注意和系统的兼容性

## 极域

如果没有杀掉极域进程，那么教师端可以看到你的屏幕画面，这就不好了。
我们可以修改极域学生端获取到的屏幕画面

有现成的工具:StudentRock

![](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/fake.PNG)
原理是向目标进程注入dll，hook GetDOBits这个函数实现自定义屏幕画面

[github](github.com/114514ns/FakeScreen)

原作者不是我，我只是修好了内存泄漏的bug。

用了几个月没有出现bug，可以放心使用。

但是它只能把屏幕画面设为静态的图片，不是特别保险，如果老师点进来，还是可以看到你的实时画面。如果有能力可以自己改进一下。

这里提供一个思路

每过一分钟，隐藏不该显示的窗口，然后截图，保存起来，然后当GetDOBits被调用时就返回刚刚保存的截图。



##  学校内网

用nmap扫了下学校内网，发现了

- 两台Vmware Exsi 服务器

- 华三交换机 telnet服务

- 几台tomcat服务器 猜测是网上阅卷的平台

- 全校的监控？

  有兴趣可以研究一下。
