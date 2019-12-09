# IO

参考大佬的博客，写得很详细：

<https://www.cnblogs.com/sxkgeek/p/9488703.html>

<https://www.cnblogs.com/restart30/p/8252109.html>

<https://blog.csdn.net/xxb2008/article/details/42424105#>

<https://blog.csdn.net/anxpp/article/details/51512200>    感觉这个代码最贴近现实应用

# BIO（同步阻塞 IO）

客户端一个请求对应一个线程。客户端上来一个请求（最开始的连接以及后续的IO请求），服务端新建一个线程去处理这个请求，由于线程总数是有限的（操作系统对线程总数的限制或者线程池的大小），所以，当达到最大值时给客户端的反馈就是无法响应，**阻塞体现在**服务端接收客户端**连接请求**被阻塞了，还有一种阻塞是在单线程处理某一个连接时，需要一直等待IO操作完成。**同步体现在**单个线程处理请求时**调用read（write）**方法需等待读取（写）操作完成才能返回。

## 缺点

1. 默认单线程有阻塞，不支持并发。（**两个阻塞**）

2. 设置成多线程的话，浪费资源。（有多少连接，就需要多少线程，并且大部分只是连接但是不做实际操作）

```java
ServerSocket server = null;
        Socket socket = null;
        InputStream inputStream = null;
        try {
            int port=8008;
            server = new ServerSocket(port);
            System.out.println("wait connection..");
            //一直监听
            while (true){
                 //阻塞一：等待连接   xxx.accept()
                socket = server.accept();
                System.out.println("connected..");
                // 建立好连接后，从socket中获取输入流，并建立缓冲区进行读取
                inputStream = socket.getInputStream();
                byte[] bytes = new byte[1024];
                int len;
                //阻塞二：等待发送内容  xx.read()
                System.out.println("wait data..");
                len = inputStream.read(bytes);
                System.out.println(new String(bytes,0,len, StandardCharsets.UTF_8));
                System.out.println("data accept complete..");
            }
        }
```



# NIO（同步非阻塞 IO）

客户端一个IO请求对应一个线程。过程是这样的，每个客户端一开始上来的连接会注册到selector中，selector会轮询注册上来的连接是否有IO请求，如果有IO请求，就创建一个线程处理该连接上的该次请求。**非阻塞体现在**服务端能够无限量（相对于BIO）的接收客户端的连接请求。**同步体现在**单个线程处理请求时**调用read（write）**方法需等待读取（写）操作完成才能返回。这种模式下，如果后端应用处理遇到资源争夺（数据库操作）而阻塞等，为提高请求的处理速度，可以在后端设立资源池或队列等，把对应的请求数据以及现场（哪个连接的哪个请求等）放入队列，前台线程立即返回处理别的IO请求。

## 改进一：

1. 单线程情况下，解阻塞。ServerSocketChannel
2. 存储链接信息，遍历是否有客户端发送消息

仍然具有缺点：性能瓶颈。连接数量超级多的时候，遍历很费时间。优化方案：由jvm交给os操作系统执行。

```java
int port = 8008;
//类似于ServerSocket，可以解阻塞
ServerSocketChannel server = ServerSocketChannel.open();
//形成端口
SocketAddress socketAddress = new InetSocketAddress(port);
//绑定端口
server.bind(socketAddress);
//设置不会阻塞
server.configureBlocking(false);
while (true){
    //获取消息
    for(SocketChannel socketChannel:list){
        int len = socketChannel.read(byteBuffer);
        //有消息
        if(len>0){
            System.out.println("=====reading"+ len);
            //把消息从buffer中读取到byte
            byteBuffer.flip();
            //注意这里byte的长度不是1024了。而是读取多少，设置多少长度。或者改为一个个字节的读取
            byte[] bytes = new byte[len];
            while (byteBuffer.remaining()>0){
                byteBuffer.get(bytes);
            }
            String message = new String(bytes, StandardCharsets.UTF_8);
            System.out.println(message);
        }else{
            //如果断开连接 应该移除
            //list.remove(socketChannel);
        }
    }
    SocketChannel accept = server.accept();
    //放入list,统一遍历去获取消息
    if(accept!=null){
        System.out.println("========加入一个连接");
        list.add(accept);
    }
}
```

## 改进二：

1. 交给底层操作系统执行（c,c++执行）

   1. 即jni实现了java调用os操作系统函数的功能。 JNI是Java Native Interface的缩写
   2. java关键字：native ，如果有native则会调用操作系统的方法。
   3. 常见的有selector,  epoll。  重点是epoll

2. 注意：不同系统源码不一样。(所以下载的时候才会有版本区分，至少是原因之一吧)

   1. windows版本的java调用的是select

   2. linux版本java调用的是epoll

```java

```

​      

## NIO方法

### ServerSocketChannel

类似于ServerSocket，重点在于提供了非阻塞的方法。configureBlocking(false).则建立的连接不是阻塞的

### SocketChannel

类似于Socket。用于客户端和服务器端的交互。

并且区别在于socket通过获取inputStream和outputStream来实现交互（理解成单向的），

但是SocketChannel是双向的。可以并且只能通过xxxBuffer实现读写。

ByteBuffer我理解的是一个容器类。里面有一个很有意思的方法：flip，用于切换读写状态。

```java
//类似于ServerSocket，可以解阻塞
ServerSocketChannel server = ServerSocketChannel.open();
//形成端口
SocketAddress socketAddress = new InetSocketAddress(port);
//绑定端口
server.bind(socketAddress);
//设置不会阻塞
server.configureBlocking(false);
//建立连接
SocketChannel accept = server.accept();
//写入到ByteBuffer
ByteBuffer byteBuffer = ByteBuffer.allocate(512);
int len = socketChannel.read(byteBuffer);
if(len>0){
    System.out.println("=====reading"+ len);
    //flip的作用是切换读写模式，由写改为读
    byteBuffer.flip();
    //注意这里byte的长度不是1024了。而是读取多少，设置多少长度。或者改为一个个字节的读取
    byte[] bytes = new byte[len];
    //判断是否还有数据，有才去取
    while (byteBuffer.remaining()>0){
        byteBuffer.get(bytes);
    }
    String message = new String(bytes, StandardCharsets.UTF_8);
    System.out.println(message);
}
```

# AIO（NIO2：异步非阻塞IO）

客户端一个IO请求对应一个线程。

过程同NIO，只是在读写IO时略有差异，对于read，方法调用后会立即返回，返回对应中有个回调方法，调用时java会告知操作系统缓冲区大小以及地址，操作系统把流里面的内容读入缓冲区后回调刚刚read返回的回调方法。对应write，方法调用后会立即返回，返回对应中有个回调方法，调用时将数据放入缓存区，操作系统往流里面写完数据后同样会回调刚刚write返回的回调方法。

**阻塞是因为此时是通过select系统调用来完成的，而select函数本身的实现方式是阻塞的，而采用select函数有个好处就是它可以同时监听多个文件句柄，从而提高系统的并发性**请求。

**异步步体现在**单个线程处理请求时**调用read（write）**方法会立即返回，返回对象有回调方法，底层OS完成IO操作后会回调该方法。

## 异步主要体现

- AsynchronousSocketChannel
- AsynchronousServerSocketChannel
- AsynchronousFileChannel
- AsynchronousDatagramChannel



# 通信协议（补充知识）

## TCP

### 优点：

 可靠，稳定 TCP的可靠体现在TCP在传递数据之前，会有三次握手来建立连接，而且在数据传递时，有确认、窗口、重传、拥塞控制机制，在数据传完后，还会断开连接用来节约系统资源。 

### 缺点： 

慢，效率低，占用系统资源高，易被攻击 TCP在传递数据之前，要先建连接，这会消耗时间，而且在数据传递时，确认机制、重传机制、拥塞控制机制等都会消耗大量的时间，而且要在每台设备上维护所有的传输连接，事实上，每个连接都会占用系统的CPU、内存等硬件资源。 而且，因为TCP有确认机制、三次握手机制，这些也导致TCP容易被人利用，实现DOS、DDOS、CC等攻击。

### 使用场景：

当对网络通讯质量有要求的时候，比如：整个数据要准确无误的传递给对方，这往往用于一些要求可靠的应用，比如HTTP、HTTPS、FTP等传输文件的协议，POP、SMTP等邮件传输的协议。 在日常生活中，常见使用TCP协议的应用如下： 浏览器，用的HTTP FlashFXP，用的FTP Outlook，用的POP、SMTP Putty，用的Telnet、SSH QQ文件传输 ………… 

## UDP

### 优点：

快，比TCP稍安全 UDP没有TCP的握手、确认、窗口、重传、拥塞控制等机制，UDP是一个无状态的传输协议，所以它在传递数据时非常快。没有TCP的这些机制，UDP较TCP被攻击者利用的漏洞就要少一些。但UDP也是无法避免攻击的，比如：UDP Flood攻击

### 缺点：

 不可靠，不稳定 因为UDP没有TCP那些可靠的机制，在数据传递时，如果网络质量不好，就会很容易丢包。

### 使用场景：

当对网络通讯质量要求不高的时候，要求网络通讯速度能尽量的快，这时就可以使用UDP。 比如，日常生活中，常见使用UDP协议的应用如下： QQ语音 QQ视频 TFTP

## RTP

是用于Internet上针对多媒体数据流的一种传输层协议。它是建立在 UDP 协议上的.
RTP 不像http和ftp可完整的下载整个影视文件，它是以固定的数据率在网络上发送数据，客户端也是按照这种速度观看影视文件，当影视画面播放过后，就不可以再重复播放

## RTCP

## RTSP

RTSP 是一种双向实时数据传输协议，它允许客户端向服务器端发送请求，如回放、快进、倒退等操作

## WebRTC

## RTMP(Real Time Messaging Protocol)

基于TCP不会丢失

## HLS

HLS 点播，基本上就是常见的分段HTTP点播，不同在于，它的分段非常小