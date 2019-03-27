---
layout:     post
title:      "ODL routed rpc 超时问题  "
date:       2019-03-27
author:     "Kingqh"
header-img: "img/post-bg-odl.jpg"
tags:
    - ODL
---



# ODL routed rpc 超时问题  

## 问题描述：

3台odl控制器组成集群，其中一台注册routed rpc服务，其他两台访问此服务。若此服务阻塞15s，则返回rpc超时，调用失败。

## 问题分析：

odl的远程通信使用的是akka机制，akka有两种消息传递方式：

- tell

异步发送一个消息并立即返回。非阻塞主线程。

	actor.tell(message, getSelf())

如果不需要对方回复，第二个参数可以为空。

- ask

异步发送一条消息并返回一个 Future代表一个可能的回应。不加回调，阻塞主线程，加回调不阻塞主线程。

	actor.ask(remoteActor, message, timeout)

使用ask将会像tell一样发送消息给接收方，接收方必须通过getSender().tell(reply, getSelf()) 发送回应来为返回的 Future 填充数据。ask 操作包括创建一个内部actor来处理回应，必须为这个内部actor指定一个超时期限，过了超时期限内部actor将被销毁以防止内存泄露。

如果一个actor 没有完成future， 它会在超时时限达到时过期， 明确作为一个参数传给ask方法，将 AskTimeoutException 填充给Future。


远程rpc调用使用的ask消息通信机制，虽然不阻塞主线程，但是odl rpc需要取到future结果，如果在ask的超时时间之内没有返回，则会出现访问失败。

![timeout]({{ site.url }}/img/in-post/2019-03-27/timeout.png)

## 解决方案

追踪odl的代码如下：

![code01]({{ site.url }}/img/in-post/2019-03-27/code01.png)

调用远程rpc时，使用 RemoteRpcImplementation 实现， invokeRpc 最终使用 RemoteDOMRpcFuture 回调 ask(remoteImplRef, executeRpcMessage, config.getAskDuration()) 。这里的 config.getAskDuration() 是从 RemoteRpcProviderConfig 对象中得到。

![code02]({{ site.url }}/img/in-post/2019-03-27/code02.png)

这里可以根据实际的业务需求，将 TAG_ASK_DURATION 参数修改为需要的超时时间。 由于 akka 通过超时时间来自动销毁 actor ，所以超时时间设置越大， 内存溢出的风险越大， 所以需要根据具体的使用场景适当增大超时时间即可。

最终通过将 TAG_ASK_DURATION 规定为 60 s，解决大多数应用场景。

![code03]({{ site.url }}/img/in-post/2019-03-27/code03.png)