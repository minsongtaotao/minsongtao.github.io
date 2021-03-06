---
layout: post
title: 讲解PHP的socket的通信原理
categories: PHP Socket
description: 自己一直不清楚网络通信的7层关系,借这个机会来学习一下.
keywords: PHP, socket的通信
---

  对TCP/IP、UDP、Socket编程这些词你不会很陌生吧？随着网络技术的发展，这些词充斥着我们的耳朵。那么我想问：

**1.         什么是TCP/IP、UDP？**

**2.         Socket在哪里呢？**

**3.         Socket是什么呢？**

**4.         你会使用它们吗？**

# 什么是TCP/IP、UDP？
  TCP/IP（Transmission Control Protocol/Internet Protocol）即传输控制协议/网间协议，是一个工业标准的协议集，它是为广域网（WANs）设计的。
  
  UDP（User Data Protocol，用户数据报协议）是与TCP相对应的协议。它是属于TCP/IP协议族中的一种。
  
 这里有一张图，表明了这些协议的关系。
 
 
![](http://onbsquc8n.bkt.clouddn.com/05172846-06817d0a3a4e412f97fa0fdf08ca6808.jpg)

# Socket在哪里呢？
　　在图1中，我们没有看到Socket的影子，那么它到底在哪里呢？还是用图来说话，一目了然。
  
  ![](http://onbsquc8n.bkt.clouddn.com/05172918-d2b39f21a08a4550b4e3c5bce482a220.jpg)
  
# Socket是什么呢？

![](http://onbsquc8n.bkt.clouddn.com/05172951-a955fce4e5d04082828e717fe0e102f9.jpg)

先从服务器端说起。服务器端先初始化Socket，然后与端口绑定(bind)，对端口进行监听(listen)，调用accept阻塞，等待客户端连接。在这时如果有个客户端初始化一个Socket，然后连接服务器(connect)，如果连接成功，这时客户端与服务器端的连接就建立了。客户端发送数据请求，服务器端接收请求并处理请求，然后把回应数据发送给客户端，客户端读取数据，最后关闭连接，一次交互结束。

# PHP的socket封装介绍说明

## socket相关函数

***

`socket_accept()` 接受一个Socket连接

`socket_bind()` 把socket绑定在一个IP地址和端口上

`socket_clear_error()` 清除socket的错误或者最后的错误代码

`socket_close()` 关闭一个socket资源

`socket_connect()` 开始一个socket连接

`socket_create_listen()` 在指定端口打开一个socket监听

`socket_create_pair()` 产生一对没有区别的socket到一个数组里

`socket_create()` 产生一个socket，相当于产生一个socket的数据结构

`socket_get_option()` 获取socket选项

`socket_getpeername()` 获取远程类似主机的ip地址

`socket_getsockname()` 获取本地socket的ip地址

`socket_iovec_add()` 添加一个新的向量到一个分散/聚合的数组

`socket_iovec_alloc()` 这个函数创建一个能够发送接收读写的iovec数据结构

`socket_iovec_delete()` 删除一个已经分配的iovec

`socket_iovec_fetch()` 返回指定的iovec资源的数据

`socket_iovec_free()` 释放一个iovec资源

`socket_iovec_set()` 设置iovec的数据新值

`socket_last_error()` 获取当前socket的最后错误代码

`socket_listen()` 监听由指定socket的所有连接

`socket_read()` 读取指定长度的数据

`socket_readv()` 读取从分散/聚合数组过来的数据

`socket_recv()` 从socket里结束数据到缓存

`socket_recvfrom()` 接受数据从指定的socket，如果没有指定则默认当前socket

`socket_recvmsg()` 从iovec里接受消息

`socket_select()` 多路选择

`socket_send()` 这个函数发送数据到已连接的socket

`socket_sendmsg()` 发送消息到socket

`socket_sendto()` 发送消息到指定地址的socket

`socket_set_block()` 在socket里设置为块模式

`socket_set_nonblock()` socket里设置为非块模式

`socket_set_option()` 设置socket选项

`socket_shutdown()` 这个函数允许你关闭读、写、或者指定的socket

`socket_strerror()` 返回指定错误号的详细错误

`socket_write()` 写数据到socket缓存

`socket_writev()` 写数据到分散/聚合数组

## 案例一：socket通信演示

服务器端:

```php

<?php
//确保在连接客户端时不会超时
set_time_limit(0);

$ip = '127.0.0.1';
$port = 16304;

/*
 +-------------------------------
 *    @socket通信整个过程
 +-------------------------------
 *    @socket_create
 *    @socket_bind
 *    @socket_listen
 *    @socket_accept
 *    @socket_read
 *    @socket_write
 *    @socket_close
 +--------------------------------
 */

/*----------------    以下操作都是手册上的    -------------------*/
if(($sock = socket_create(AF_INET,SOCK_STREAM,SOL_TCP)) < 0) {
    echo "socket_create() 失败的原因是:".socket_strerror($sock)."\n";
}

if(($ret = socket_bind($sock,$ip,$port)) < 0) {
    echo "socket_bind() 失败的原因是:".socket_strerror($ret)."\n";
}

if(($ret = socket_listen($sock,4)) < 0) {
    echo "socket_listen() 失败的原因是:".socket_strerror($ret)."\n";
}

$count = 0;

do {
    if (($msgsock = socket_accept($sock)) < 0) {
        echo "socket_accept() failed: reason: " . socket_strerror($msgsock) . "\n";
        break;
    } else {
        
        //发到客户端
        $msg ="测试成功！\n";
        socket_write($msgsock, $msg, strlen($msg));
        
        echo "测试成功了啊\n";
        $buf = socket_read($msgsock,8192);
        
        
        $talkback = "收到的信息:$buf\n";
        echo $talkback;
        
        if(++$count >= 5){
            break;
        };
        
    
    }
    //echo $buf;
    socket_close($msgsock);

} while (true);

socket_close($sock);

```

这是`socket`的服务端代码

然后在`linux`环境上运行

![](http://onbsquc8n.bkt.clouddn.com/%E5%90%AF%E5%8A%A8socket%E6%9C%8D%E5%8A%A1%E5%99%A8.bmp)

没有反应, 对于服务器端的程序已经开始运行

运行`nmap -p 16300-16309 localhost` 我们的是16304端口

![](http://onbsquc8n.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20170331101647.bmp)

可以看到端口处于开启状态, 我们运行client 的程序即可以连接上

```php

<?php
error_reporting(E_ALL);
set_time_limit(0);
echo "<h2>TCP/IP Connection</h2>\n";

$port = 16304;
$ip = "127.0.0.1";

/*
 +-------------------------------
 *    @socket连接整个过程
 +-------------------------------
 *    @socket_create
 *    @socket_connect
 *    @socket_write
 *    @socket_read
 *    @socket_close
 +--------------------------------
 */

$socket = socket_create(AF_INET, SOCK_STREAM, SOL_TCP);
if ($socket < 0) {
    echo "socket_create() failed: reason: " . socket_strerror($socket) . "\n";
}else {
    echo "OK.\n";
}

echo "试图连接 '$ip' 端口 '$port'...\n";
$result = socket_connect($socket, $ip, $port);
if ($result < 0) {
    echo "socket_connect() failed.\nReason: ($result) " . socket_strerror($result) . "\n";
}else {
    echo "连接OK\n";
}

$in = "Ho\r\n";
$in .= "first blood\r\n";
$out = '';

if(!socket_write($socket, $in, strlen($in))) {
    echo "socket_write() failed: reason: " . socket_strerror($socket) . "\n";
}else {
    echo "发送到服务器信息成功！\n";
    echo "发送的内容为:<font color='red'>$in</font> <br>";
}

while($out = socket_read($socket, 8192)) {
    echo "接收服务器回传信息成功！\n";
    echo "接受的内容为:",$out;
}


echo "关闭SOCKET...\n";
socket_close($socket);
echo "关闭OK\n";

```

![](http://onbsquc8n.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BEsocket%E5%AE%A2%E6%88%B7%E7%AB%AF.bmp)

![](http://onbsquc8n.bkt.clouddn.com/socket%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%AB%AF.bmp)

至此客户端已经连接上服务端了

## 案例二：代码详解

```php
// 设置一些基本的变量
$host = "192.168.1.99";
$port = 1234;
// 设置超时时间
set_time_limit(0);
// 创建一个Socket
$socket = socket_create(AF_INET, SOCK_STREAM, 0) or die("Could not createsocket\n");
//绑定Socket到端口
$result = socket_bind($socket, $host, $port) or die("Could not bind tosocket\n");
// 开始监听链接
$result = socket_listen($socket, 3) or die("Could not set up socketlistener\n");
// accept incoming connections
// 另一个Socket来处理通信
$spawn = socket_accept($socket) or die("Could not accept incomingconnection\n");
// 获得客户端的输入
$input = socket_read($spawn, 1024) or die("Could not read input\n");
// 清空输入字符串
$input = trim($input);
//处理客户端输入并返回结果
$output = strrev($input) . "\n";
socket_write($spawn, $output, strlen ($output)) or die("Could not write
output\n");
// 关闭sockets
socket_close($spawn);
socket_close($socket);

下面是其每一步骤的详细说明:

1.第一步是建立两个变量来保存Socket运行的服务器的IP地址和端口.你可以设置为你自己的服务器和端口(这个端口可以是1到65535之间的数字),前提是这个端口未被使用.

[Copy to clipboard]
PHP CODE:

// 设置两个变量
$host = "192.168.1.99";
$port = 1234;

2.在服务器端可以使用set_time_out()函数来确保PHP在等待客户端连接时不会超时.

[Copy to clipboard]
PHP CODE:

// 超时时间
set_time_limit(0);

3.在前面的基础上,现在该使用socket_creat()函数创建一个Socket了—这个函数返回一个Socket句柄,这个句柄将用在以后所有的函数中.

[Copy to clipboard]
PHP CODE:

// 创建Socket
$socket = socket_create(AF_INET, SOCK_STREAM, 0) or die("Could not create
socket\n");

第一个参数”AF_INET”用来指定域名;
第二个参数”SOCK_STREM”告诉函数将创建一个什么类型的Socket(在这个例子中是TCP类型)

因此,如果你想创建一个UDP Socket的话,你可以使用如下的代码:

[Copy to clipboard]
PHP CODE:

// 创建 socket
$socket = socket_create(AF_INET, SOCK_DGRAM, 0) or die("Could not create
socket\n");

4.一旦创建了一个Socket句柄,下一步就是指定或者绑定它到指定的地址和端口.这可以通过socket_bind()函数来完成.

[Copy to clipboard]
PHP CODE:

// 绑定 socket to 指定地址和端口
$result = socket_bind($socket, $host, $port) or die("Could not bind to
socket\n");

5.当Socket被创建好并绑定到一个端口后,就可以开始监听外部的连接了.PHP允许你由socket_listen()函数来开始一个监听,同时你可以指定一个数字(在这个例子中就是第二个参数:3)

[Copy to clipboard]
PHP CODE:

// 开始监听连接
$result = socket_listen($socket, 3) or die("Could not set up socket
listener\n");

6.到现在,你的服务器除了等待来自客户端的连接请求外基本上什么也没有做.一旦一个客户端的连接被收到,socket_accept()函数便开始起作用了,它接收连接请求并调用另一个子Socket来处理客户端–服务器间的信息.

[Copy to clipboard]
PHP CODE:

//接受请求链接
// 调用子socket 处理信息
$spawn = socket_accept($socket) or die("Could not accept incoming
connection\n");

这个子socket现在就可以被随后的客户端–服务器通信所用了.

7.当一个连接被建立后,服务器就会等待客户端发送一些输入信息,这写信息可以由socket_read()函数来获得,并把它赋值给PHP的$input变量.

[Copy to clipboard]
PHP CODE:

// 读取客户端输入
$input = socket_read($spawn, 1024) or die("Could not read input\n");
?&gt;

socker_read的第而个参数用以指定读入的字节数,你可以通过它来限制从客户端获取数据的大小.

注意:socket_read函数会一直读取壳户端数据,直到遇见\n,\t或者\0字符.PHP脚本把这写字符看做是输入的结束符.

8.现在服务器必须处理这些由客户端发来是数据(在这个例子中的处理仅仅包含数据的输入和回传到客户端).这部分可以由socket_write()函数来完成(使得由通信socket发回一个数据流到客户端成为可能)

[Copy to clipboard]
PHP CODE:

// 处理客户端输入并返回数据
$output = strrev($input) . "\n";
socket_write($spawn, $output, strlen ($output)) or die("Could not write
output\n");

9.一旦输出被返回到客户端,父/子socket都应通过socket_close()函数来终止

[Copy to clipboard]
PHP CODE:

// 关闭 sockets
socket_close($spawn);
socket_close($socket);

```




