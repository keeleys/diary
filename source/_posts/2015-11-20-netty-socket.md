title: netty5实现简单的socket服务器
layout: post
comments: true
keywords: netty_socket
description: netty_socket
categories: 
	-netty
	-编程记录
date: 2015-11-20 10:06:18
tag: netty
---

## 安装
```xml
<dependency>
	<groupId>io.netty</groupId>
	<artifactId>netty-all</artifactId>
	<version>5.0.0.Alpha2</version>
</dependency>
```
<!-- more -->

## 代码
* NettyService
``` java

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.*;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import io.netty.util.CharsetUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Created by jun on 2015/11/2.
 */
public class NettyService {
    public static final int THREADSIZE = 10;
   // public static final int port = 9019;
    private static Logger log = LoggerFactory.getLogger(NettyService.class);
    private static EventLoopGroup bossGroup = null;
    private static EventLoopGroup workerGroup = null;
    public static void start(int port) throws InterruptedException {
        ServerBootstrap sb = null;
        try {
            bossGroup = new NioEventLoopGroup();
            workerGroup = new NioEventLoopGroup(THREADSIZE);

            sb = new ServerBootstrap();
            sb.option(ChannelOption.SO_BACKLOG, 1024);
            sb.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .handler(new LoggingHandler(LogLevel.INFO))
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ChannelPipeline p = ch.pipeline();
                            p.addLast("decoder", new StringDecoder(CharsetUtil.UTF_8));
                           // p.addLast("encoder", new StringEncoder(CharsetUtil.UTF_8));
                            p.addLast(new NettyServerHandler());
                        }

                    });
            Channel ch = sb.bind(port).sync().channel();
            log.info("server 127.0.0.1:" + port + " is starting");
            ch.closeFuture().sync();//.addListener(new WebSocketListener());
        }finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }


    }

    public static void stop() {
        bossGroup.shutdownGracefully();
        workerGroup.shutdownGracefully();
    }
}

```

* NettyServerHandler
```java
package com.uxun.interfaces.netty;

import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerAdapter;
import io.netty.channel.ChannelHandlerContext;
import io.netty.util.ReferenceCountUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.io.UnsupportedEncodingException;

/**
 * Created by jun on 2015/11/2.
 */
public class NettyServerHandler extends ChannelHandlerAdapter {
    public static final Logger log = LoggerFactory.getLogger(NettyServerHandler.class);
    // 接收到新的数据
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws UnsupportedEncodingException {
        try {
            String str = msg.toString();
            NettyProcess process = new NettyProcess();//一个返回str的操作.
            String result = process.process(str);
            // 发送到客户端
            byte[] responseByteArray = result.getBytes("UTF-8");
            ByteBuf out = ctx.alloc().buffer(responseByteArray.length);
            out.writeBytes(responseByteArray);
            ctx.writeAndFlush(out);
           
        } catch (IOException e) {
            log.error("netty服务器出错",e);
            e.printStackTrace();
        } finally {
            ReferenceCountUtil.release(msg);
            ctx.close();
        }
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) {
        ctx.flush();
    }
    @Override
    public void channelActive(ChannelHandlerContext ctx) {
        System.out.println("channelActive");
    }

    @Override
    public void channelInactive(ChannelHandlerContext ctx){
        System.out.println("channelInactive");
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        // Close the connection when an exception is raised.
        cause.printStackTrace();
        ctx.close();
    }
}

```

## 参考
[http://www.cnblogs.com/wucao/p/3934913.html](http://www.cnblogs.com/wucao/p/3934913.html)