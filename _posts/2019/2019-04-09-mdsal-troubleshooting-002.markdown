---
layout:     post
title:      "ODL routed rpc 失败返回问题"
date:       2019-04-09
author:     "Kingqh"
header-img: "img/post-bg-odl.jpg"
tags:
    - ODL
---

# ODL routed rpc 失败返回问题  #

## 问题描述：

3台odl控制器组成集群，其中一台注册routed rpc服务，其他两台访问此服务。若对应的rpc服务的返回值为error，返回的错误信息和在同节点访问返回的结果不同。不同的返回信息如下：

访问节点和服务注册节点相同时，返回：

	{
	    "errors": {
	        "error": [
	            {
	                "error-type": "application",
	                "error-tag": "data-missing",
	                "error-message": "The tunnel0/0/100 does not exist.",
	                "error-info": "<mpls>\n        \n          \n            \n              tunnel0/0/100\n            \n          \n        \n      </mpls><error-paras>\n        tunnel0/0/100\n      </error-paras>"
	            }
	        ]
	    }
	}

访问节点和服务注册节点不同时，返回：

	{
	    "errors": {
	        "error": [
	            {
	                "error-type": "application",
	                "error-tag": "operation-failed",
	                "error-message": "The operation encountered an unexpected error while executing.",
	                "error-info": "org.opendaylight.controller.remote.rpc.RpcErrorsException: Execution of RPC (com:tencent:teg:dcic:netconf:rsvp?revision=2018-06-04)delete-rsvp-tunnel failed\n\tat org.opendaylight.controller.remote.rpc.RpcBroker$1.onSuccess(RpcBroker.java:82)\n\tat org.opendaylight.controller.remote.rpc.RpcBroker$1.onSuccess(RpcBroker.java:72)\n\tat com.google.common.util.concurrent.Futures$6.run(Futures.java:1319)\n\tat com.google.common.util.concurrent.MoreExecutors$DirectExecutor.execute(MoreExecutors.java:457)\n\tat com.google.common.util.concurrent.ExecutionList.executeListener(ExecutionList.java:156)\n\tat com.google.common.util.concurrent.ExecutionList.execute(ExecutionList.java:145)\n\tat com.google.common.util.concurrent.AbstractFuture.set(AbstractFuture.java:185)\n\tat com.google.common.util.concurrent.SettableFuture.set(SettableFuture.java:53)\n\tat com.tencent.teg.dcic.netconf.impl.RsvpServiceImpl$10.onSuccess(RsvpServiceImpl.java:434)\n\tat com.tencent.teg.dcic.netconf.impl.RsvpServiceImpl$10.onSuccess(RsvpServiceImpl.java:427)\n\tat com.google.common.util.concurrent.Futures$6.run(Futures.java:1319)\n\tat com.google.common.util.concurrent.MoreExecutors$DirectExecutor.execute(MoreExecutors.java:457)\n\tat com.google.common.util.concurrent.ExecutionList.executeListener(ExecutionList.java:156)\n\tat com.google.common.util.concurrent.ExecutionList.execute(ExecutionList.java:145)\n\tat com.google.common.util.concurrent.AbstractFuture.set(AbstractFuture.java:185)\n\tat com.google.common.util.concurrent.Futures$ChainingListenableFuture$1.run(Futures.java:918)\n\tat com.google.common.util.concurrent.MoreExecutors$DirectExecutor.execute(MoreExecutors.java:457)\n\tat com.google.common.util.concurrent.Futures$ImmediateFuture.addListener(Futures.java:106)\n\tat com.google.common.util.concurrent.Futures$ChainingListenableFuture.run(Futures.java:914)\n\tat com.google.common.util.concurrent.MoreExecutors$DirectExecutor.execute(MoreExecutors.java:457)\n\tat com.google.common.util.concurrent.ExecutionList.executeListener(ExecutionList.java:156)\n\tat com.google.common.util.concurrent.ExecutionList.execute(ExecutionList.java:145)\n\tat com.google.common.util.concurrent.AbstractFuture.set(AbstractFuture.java:185)\n\tat org.opendaylight.netconf.sal.connect.netconf.listener.UncancellableFuture.set(UncancellableFuture.java:44)\n\tat org.opendaylight.netconf.sal.connect.netconf.listener.NetconfDeviceCommunicator.processMessage(NetconfDeviceCommunicator.java:310)\n\tat org.opendaylight.netconf.sal.connect.netconf.listener.NetconfDeviceCommunicator.onMessage(NetconfDeviceCommunicator.java:257)\n\tat org.opendaylight.netconf.sal.connect.netconf.listener.NetconfDeviceCommunicator.onMessage(NetconfDeviceCommunicator.java:48)\n\tat org.opendaylight.netconf.nettyutil.AbstractNetconfSession.handleMessage(AbstractNetconfSession.java:64)\n\tat org.opendaylight.netconf.nettyutil.AbstractNetconfSession.handleMessage(AbstractNetconfSession.java:35)\n\tat org.opendaylight.protocol.framework.AbstractProtocolSession.channelRead0(AbstractProtocolSession.java:53)\n\tat io.netty.channel.SimpleChannelInboundHandler.channelRead(SimpleChannelInboundHandler.java:105)\n\tat io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:342)\n\tat io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:328)\n\tat io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:321)\n\tat io.netty.handler.codec.ByteToMessageDecoder.fireChannelRead(ByteToMessageDecoder.java:293)\n\tat io.netty.handler.codec.ByteToMessageDecoder.channelRead(ByteToMessageDecoder.java:267)\n\tat io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:342)\n\tat io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:328)\n\tat io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:321)\n\tat io.netty.handler.codec.ByteToMessageDecoder.fireChannelRead(ByteToMessageDecoder.java:293)\n\tat io.netty.handler.codec.ByteToMessageDecoder.channelRead(ByteToMessageDecoder.java:267)\n\tat io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:342)\n\tat io.netty.channel.AbstractChannelHandlerContext.access$600(AbstractChannelHandlerContext.java:33)\n\tat io.netty.channel.AbstractChannelHandlerContext$7.run(AbstractChannelHandlerContext.java:333)\n\tat io.netty.util.concurrent.SingleThreadEventExecutor.runAllTasks(SingleThreadEventExecutor.java:358)\n\tat io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:394)\n\tat io.netty.util.concurrent.SingleThreadEventExecutor$2.run(SingleThreadEventExecutor.java:112)\n\tat io.netty.util.concurrent.DefaultThreadFactory$DefaultRunnableDecorator.run(DefaultThreadFactory.java:145)\n\tat java.lang.Thread.run(Thread.java:748)\n"
	            }
	        ]
	    }
	}

## 解决方案：

修改代码 controller-release-beryllium-sr4/opendaylight/md-sal/sal-remoterpc-connector/src/main/java/org/opendaylight/controller/remote/rpc/RemoteDOMRpcFuture.java 如下：

	     private final class FutureUpdater extends OnComplete<Object> {
	-
	-        @Override
	+       
	+       @Override
	         public void onComplete(final Throwable error, final Object reply) throws Throwable {
	             if (error != null) {
	-                RemoteDOMRpcFuture.this.failNow(error);
	+                if (error instanceof RpcErrorsException) {
	+                    RemoteDOMRpcFuture.this.set(new DefaultDOMRpcResult(((RpcErrorsException) error).getRpcErrors()));
	+                } else {
	+                    RemoteDOMRpcFuture.this.failNow(error);
	+                }
	             } else if (reply instanceof RpcResponse) {
	                 final RpcResponse rpcReply = (RpcResponse) reply;
	                 final NormalizedNode<?, ?> result;
	@@ -102,10 +106,11 @@ class RemoteDOMRpcFuture extends AbstractFuture<DOMRpcResult> implements Checked
	                 }
	                 RemoteDOMRpcFuture.this.set(new DefaultDOMRpcResult(result));
	                 LOG.debug("Future {} for rpc {} successfully completed", RemoteDOMRpcFuture.this, rpcName);
	+            } else {
	+                RemoteDOMRpcFuture.this.failNow(new IllegalStateException("Incorrect reply type " + reply
	+                        + "from Akka"));
	             }
	-            RemoteDOMRpcFuture.this.failNow(new IllegalStateException("Incorrect reply type " + reply
	-                    + "from Akka"));
	-        }
	+        }      
	     }
	 
	 }

routed rpc返回的异常若为 RpcErrorsException 则说明远端节点rpc运行成功，但是RpcResult中有error信息。


![rpcBroker](/img/in-post/2019-04-09/rpcBroker.png)

有上图可以看出，Future的success和RpcResult的success区别方法不一样，前者只能代表业务是否正常运行，后者则是业务运行成功后返回的结果封装。ODL对routed rpc将Future成功, rpcResult error不为空的处理和local rpc不同。导致两者返回错误信息不一致，那么则无法从error中提取特定的信息。

经过代码修改，能够实现访问routed rpc和local rpc返回结果一直。
