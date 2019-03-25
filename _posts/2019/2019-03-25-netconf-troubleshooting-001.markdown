---
layout:     post
title:      "一个Netconf问题解决"
date:       2019-03-25
author:     "Kingqh"
header-img: "img/post-bg-odl-structure.jpg"
tags:
    - ODL
---


# 一个Netconf问题解决 #

## 问题描述：

在设备上未配置netconf over ssh,但是使用ODL控制器反复尝试和设备建立netconf会话，尝试建连得端口号和ssh默认端口一样都是22。每次尝试建连不成功，启动新的tcp会话，旧的会话未删除，导致设备上的会话愈来愈多，直到达到设备tcp会话上限。最后使用ssh无法再登录设备。

![aaa]({{ site.url }}/img/in-post/2019-03-25/01.png)

## 原因猜测：

由于netconf是基于ssh协议加密认证之后建立的会话，从现象看，可能是ssh会话建立成功后，netconf会话建立失败，server(设备)和控制器没有协商拆除ssh会话。ssh是基于tcp协议建立的应用，所以ssh会话的建立和拆除同时会导致tcp会话的建立和拆除。使用抓包查看，出现问题的过程中没有发现拆除tcp连接的FIN报文。下图是正常tcp断链报文交互：

![bbb]({{ site.url }}/img/in-post/2019-03-25/02.png)

## 代码修改

跟踪代码查看，可以看到，AbstractNetconfSessionNegotiator 类中执行到ssh会话建立成功，开始协商Netconf能力集建立Netconf会话。但由于设备上未使能Netconf协议，所以能力集协商超时。

如图日志所示：

![log]({{ site.url }}/img/in-post/2019-03-25/log.png)

协商超时代码如下：

![code0]({{ site.url }}/img/in-post/2019-03-25/code0.png)

从上面两个图片可以看出，控制器认为netconf会话的channel关闭成功，但是实际上ssh tcp未关闭成功。

追踪到ssh会话关闭代码如下：

![code1]({{ site.url }}/img/in-post/2019-03-25/code1.png)

由于设备实现不同，导致ssh会话平滑关闭失败。此时需要使用强制关闭功能。修改代码如下：


![code2]({{ site.url }}/img/in-post/2019-03-25/code2.png)

当使用平滑关闭会话失败后，启用强制会话关闭。虽然比较暴力，但是解决了问题，经过测试和验证，目前没有发现问题。而这段代码得作者在回调时，若会话没有关闭，则强制关闭。但没有考虑到有可能回调无法被执行到这种情况，此时会话会永久残留。

