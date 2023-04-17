# Netty笔记

[TOC]

## 1. OverView

### 1.1 介绍

`Netty`是一个异步事件驱动的网络应用程序框架用于快速开发可维护的高性能协议服务器和客户端, 是一个`NIO`异步非阻塞应用程序框架。

`Netty`是`jboss`提供的一个`java`开源框架，`netty`提供异步的、事件驱动的网络应用程序框架和工具，用以快速开发高性能、高可用性的网络服务器和客户端程序。也就是说netty是一个基于`nio`的编程框架，使用`netty`可以快速的开发出一个网络应用。

### 1.2 架构思想

`Netty`的核心是支持零拷贝的bytebuf缓冲对象、通用通信api和可扩展的事件模型；它支持多种传输服务并且支持`HTTP`、`Protobuf`、二进制、文本、`WebSocket` 等一系列常见协议，也支持自定义协议。

![Image](https://netty.io/images/components.png)

更多`Netty的`概念请参考[Netty基础介绍](https://blog.csdn.net/weixin_44688301/article/details/116195049)， [官网地址](https://netty.io)

### 1.3 配置

- Jar包下载： https://netty.io/downloads.html；https://mvnrepository.com/artifact/io.netty/netty-all；

或直接根据需要`Groovy`配置：

```groovy
// for netty lib
implementation group: 'io.netty', name: 'netty-transport', version: '4.1.59.Final'
implementation group: 'io.netty', name: 'netty-resolver', version: '4.1.59.Final'
implementation group: 'io.netty', name: 'netty-handler', version: '4.1.59.Final'
implementation group: 'io.netty', name: 'netty-common', version: '4.1.59.Final'
implementation group: 'io.netty', name: 'netty-codec', version: '4.1.59.Final'
implementation group: 'io.netty', name: 'netty-buffer', version: '4.1.59.Final'

//配置全部包
implementation group: 'io.netty', name: 'netty-all', version: '4.1.59.Final'

```

> 注： 记得添加网络权限
>
> ```groovy
> <uses-permission android:name="android.permission.INTERNET" />
> ```

## 2. 基础使用

### 2.1 实现服务端代码

- 创建`NettyServer.java`开启`tcp`服务

  ```java
  public class NettyServer {
      private static final String TAG = "NettyServer";
  
      //端口
      private static final int PORT = 7010;
  
      /**
       * 启动tcp服务端
       */
      public void startServer() {
          try {
              // 创建mainReactor
              EventLoopGroup bossGroup = new NioEventLoopGroup();
              // 创建工作线程组
              EventLoopGroup workerGroup = new NioEventLoopGroup();
              ServerBootstrap b = new ServerBootstrap();
              b.group(bossGroup, workerGroup)// 组装NioEventLoopGroup
                  // 设置channel类型为NIO类型
                  .channel(NioServerSocketChannel.class)
                  // 设置连接配置参数
                  .option(ChannelOption.SO_BACKLOG, 1024)
                  .childOption(ChannelOption.SO_KEEPALIVE, true)
                  .childOption(ChannelOption.TCP_NODELAY, true)
                  // 配置入站、出站事件handler
                  .childHandler(new ChannelInitializer<SocketChannel>() {
                      @Override
                      protected void initChannel(SocketChannel socketChannel) throws Exception {
                          ChannelPipeline pipeline = socketChannel.pipeline();
                          //添加发送数据编码器
                          pipeline.addLast(new ServerEncoder());
                          //添加解码器，对收到的数据进行解码
                          pipeline.addLast(new ServerDecoder());
                          //添加数据处理
                          pipeline.addLast(new ServerHandler());
                      }
                  });
              //服务器启动辅助类配置完成后，调用 bind 方法绑定监听端口，调用 sync 方法同步等待绑定操作完成
              b.bind(PORT).sync();
              Log.d(TAG, "TCP 服务启动成功 PORT = " + PORT);
          } catch (Exception e) {
              e.printStackTrace();
          }
      }
  }
  
  ```

- `ServerEncoder.java`服务端发送数据编码器

  ```java
  public class ServerEncoder extends MessageToByteEncoder<Object> {
      private static final String TAG = "ServerEncoder";
  
      @Override
      protected void encode(ChannelHandlerContext channelHandlerContext, Object data, ByteBuf byteBuf) throws Exception {
          //自己发送过来的东西进行编码
          byteBuf.writeBytes(data.toString().getBytes());
      }
  }
  ```

- `ServerDecoder.java`解码器，对客户端的数据进行解析

  ```java
  public class ServerDecoder extends ByteToMessageDecoder {
      private static final String TAG = "ServerDecoder";
  
      @Override
      protected void decode(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf, List<Object> list) throws Exception {
          //收到的数据长度
          int length = byteBuf.readableBytes();
          //创建个byteBuf存储数据，进行编辑
          ByteBuf dataBuf = Unpooled.buffer(length);
          //写入收到的数据
          dataBuf.writeBytes(byteBuf);
          //将byteBuf转为数组
          String data = new String(dataBuf.array());
          Log.d(TAG, "收到了客户端发送的数据：" + data);
          //将数据传递给下一个Handler，也就是在NettyServer给ChannelPipeline添加的处理器
          list.add(data);
      }
  }
  ```

- `ServerHandler.java`数据处理器

  ```java
  public class ServerHandler extends SimpleChannelInboundHandler<Object> {
  
      private static final String TAG = "ServerHandler";
  
      /**
       * 当收到数据的回调
       *
       * @param channelHandlerContext 封装的连接对像
       * @param o
       * @throws Exception
       */
      @Override
      protected void channelRead0(ChannelHandlerContext channelHandlerContext, Object o) throws Exception {
          Log.d(TAG, "收到了解码器处理过的数据：" + o.toString());
      }
  
      /**
       * 有客户端连接过来的回调
       *
       * @param ctx
       * @throws Exception
       */
      @Override
      public void channelActive(ChannelHandlerContext ctx) throws Exception {
          super.channelActive(ctx);
          Log.d(TAG, "有客户端连接过来：" + ctx.toString());
      }
  
      /**
       * 有客户端断开了连接的回调
       *
       * @param ctx
       * @throws Exception
       */
      @Override
      public void channelInactive(ChannelHandlerContext ctx) throws Exception {
          super.channelInactive(ctx);
          Log.d(TAG, "有客户端断开了连接：" + ctx.toString());
      }
  }
  ```

  基本过程如下：

  - 初始化创建2个NioEventLoopGroup，其中boosGroup用于Accetpt连接建立事件并分发请求， workerGroup用于处理I/O读写事件和业务逻辑

  - 基于ServerBootstrap(服务端启动引导类)，配置EventLoopGroup、Channel类型，连接参数、配置入站、出站事件handler

### 2.2 实现客户端代码

- 创建`NettyClient.java`

  ```java
  public class NettyClient {
      private static final String TAG = "NettyClient";
      private final int PORT = 7010;
      //连接的服务端ip地址
      private final String IP = "192.168.3.111";
      private static NettyClient nettyClient;
      //与服务端的连接通道
      private Channel channel;
      
      public static NettyClient getInstance() {
          if (nettyClient == null) {
              nettyClient = new NettyClient();
          }
          return nettyClient;
      }
      /**
       *需要在子线程中发起连接
       */
      private NettyClient() {
          new Thread(new Runnable() {
              @Override
              public void run() {
                  connect();
              }
          }).start();
      }
      /**
       * 获取与服务端的连接
       */
      public static Channel getChannel() {
          if (nettyClient == null) {
              return null;
          }
          return nettyClient.channel;
      }
      /**
       * 连接服务端
       */
      public void connect() {
          try {
              NioEventLoopGroup group = new NioEventLoopGroup();//线程组：用来进行网络通讯读写
              Bootstrap bootstrap = new Bootstrap()
                      // 指定channel类型
                      .channel(NioSocketChannel.class)
                      // 指定EventLoopGroup
                      .group(group)
                      .option(ChannelOption.SO_KEEPALIVE, true)
                      // 指定Handler
                      .handler(new ChannelInitializer<SocketChannel>() {
                          @Override
                          protected void initChannel(SocketChannel socketChannel) throws Exception {
                              ChannelPipeline pipeline = socketChannel.pipeline();
                              //添加发送数据编码器
                              pipeline.addLast(new ClientEncoder());
                              //添加数据处理器
                              pipeline.addLast(new ClientHandler());
                          }
                      });
              // 连接到服务端
              ChannelFuture channelFuture = bootstrap.connect(new InetSocketAddress(IP, PORT));
              //获取连接通道
              channel = channelFuture.sync().channel();
          } catch (Exception e) {
              Log.e(TAG, "连接失败：" + e.getMessage());
              e.printStackTrace();
          }
      }
  }
  ```

- `ClientEncoder.java`对发送的数据进行编码

  ```java
  public class ClientEncoder extends MessageToByteEncoder<Object> {
  
      private static final String TAG = "ClientEncoder";
  
      @Override
      protected void encode(ChannelHandlerContext channelHandlerContext, Object data, ByteBuf byteBuf) throws Exception {
          //自己发送过来的东西进行编码
          byteBuf.writeBytes(data.toString().getBytes());
      }
  }
  ```

- `ClientHandler.java`数据处理器

  ```java
  public class ClientHandler extends SimpleChannelInboundHandler<Object> {
  
      private static final String TAG = "ClientHandler";
  
      /**
       * 当收到数据的回调
       *
       * @param channelHandlerContext 封装的连接对像
       * @param o
       * @throws Exception
       */
      @Override
      protected void channelRead0(ChannelHandlerContext channelHandlerContext, Object o) throws Exception {
  		System.out.println(o);
          //此处使该方法中管道和信息往下传播到下一个处理类
          //ctx.fireChannelRead(o);
      }
  
      /**
       * 与服务端连接成功的回调
       *
       * @param ctx
       * @throws Exception
       */
      @Override
      public void channelActive(ChannelHandlerContext ctx) throws Exception {
          super.channelActive(ctx);
          Log.d(TAG, "与服务端连接成功：" + ctx.toString());
      }
  
      /**
       * 与服务端断开的回调
       *
       * @param ctx
       * @throws Exception
       */
      @Override
      public void channelInactive(ChannelHandlerContext ctx) throws Exception {
          super.channelInactive(ctx);
          Log.d(TAG, "与服务端断开连接：" + ctx.toString());
      }
  }
  ```

  





https://azhon.blog.csdn.net/article/details/100569489

https://blog.csdn.net/weixin_44688301/article/details/116195049

https://blog.csdn.net/weixin_42408447/article/details/117110363