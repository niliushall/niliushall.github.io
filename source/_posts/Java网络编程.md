---
title: Java网络编程
date: 2023-03-27 22:13:27
description: 本文主要介绍Java网络编程相关基础知识。
tags:
    - Java
    - 网络编程
categories:
    - Java
cover: http://cdn-hw-static.shanhutech.cn/bizhi/staticwp/202301/9c3d7292c468436541f121da5cbe1aec--964716261.jpg
---

# 网络编程基础

## 1. IP地址

InetAddress基础用法

```java
package pers.wl.netprograming;

import java.net.InetAddress;
import java.net.UnknownHostException;

public class TestInetAddress {
    public static void main(String[] args) {
        try {
            InetAddress inetAddress1 = InetAddress.getByName("localhost");
            System.out.println(inetAddress1);

            InetAddress inetAddress2 = InetAddress.getLocalHost();
            System.out.println(inetAddress2);

            InetAddress inetAddress3 = InetAddress.getByName("www.baidu.com");
            System.out.println(inetAddress3);

            // 常用方法
            System.out.println(inetAddress3.getCanonicalHostName());
            System.out.println(inetAddress3.getHostAddress());
            System.out.println(inetAddress3.getHostName());
        } catch (UnknownHostException e) {
            e.printStackTrace();
        }
    }
}
```

## 2. 端口

端口范围：`0~65535`，其中：

- `0~1023`作为公有端口
  - HTTP：80
  - HTTPS：443
  - FTP：21
  - Telnet：23
- `1024~49151`作为程序注册端口
  - Tomcat：8080
  - MySQL：3306
  - Oracle：1521
- 剩余端口为动态端口
  - 查看所有端口：`netstat -ano`

```java
package pers.wl.netprograming;

import java.net.InetSocketAddress;

public class TestInetSocketAddress {
    public static void main(String[] args) {
        InetSocketAddress inetSocketAddress = new InetSocketAddress("www.baidu.com", 8080);
        System.out.println(inetSocketAddress.getAddress());
    }
}
```

## 3. TCP实现聊天

### 3.1 服务器端

步骤：

- 创建`ServerSocket`
- 调用`accept`等待客户端连接，并生成`Socket`对象
- 对`socket`对象调用`getInputStream`以获取输入流
- 利用`ByteArrayOutputStream`与字节数组读取数据，`ByteArrayOutputStream`可以保证数据不会乱码
- 最后关闭相关资源

```java
package pers.wl.netprograming;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.net.InetAddress;
import java.net.ServerSocket;
import java.net.Socket;

public class Server {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = null;
        Socket socket = null;
        InputStream is = null;
        ByteArrayOutputStream baos = null;

        try {
            serverSocket = new ServerSocket(9999);
            socket = serverSocket.accept();
            is = socket.getInputStream();

            baos = new ByteArrayOutputStream();
            byte[] buffer = new byte[1024];
            int len = 0;
            while ((len = is.read(buffer)) != -1) {
                baos.write(buffer, 0, len);
            }

            System.out.println(baos.toString());
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (baos != null) {
                baos.close();
            }
            if (is != null) {
                is.close();
            }
            if (socket != null) {
                socket.close();
            }
            if (serverSocket != null) {
                serverSocket.close();
            }
        }
    }
}
```

### 3.2 客户端

步骤：

- 定义IP、port
- 创建`Socket`，并调用`getOutputStream`获取输出流，通过`write`写数据（PS：需要使用`getBytes`将数据转换为字节流）

```java
package pers.wl.netprograming;

import java.io.IOException;
import java.io.OutputStream;
import java.net.InetAddress;
import java.net.Socket;
import java.net.UnknownHostException;

public class Client {
    public static void main(String[] args) throws IOException {
        Socket socket = null;
        OutputStream os = null;

        try {
            InetAddress serverIP = InetAddress.getByName("127.0.0.1");
            int port = 9999;

            socket = new Socket(serverIP, port);
            os = socket.getOutputStream();
            os.write("测试通信".getBytes());
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (os != null) {
                os.close();
            }
            if (socket != null) {
                socket.close();
            }
        }
    }
}
```

## 4. TCP实现文件传输

### 4.1 服务器端

```java
package pers.wl.netprograming;

import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;

public class FileServer {
    public static void main(String[] args) throws Exception {
        ServerSocket serverSocket = new ServerSocket(9000);
        Socket socket = serverSocket.accept();
        InputStream inputStream = socket.getInputStream();

        FileOutputStream fos = new FileOutputStream(new File("receive.png"));
        byte[] buffer = new byte[1024];
        int len = 0;

        while ((len = inputStream.read(buffer)) != -1) {
            fos.write(buffer, 0, len);
        }

        OutputStream outputStream = socket.getOutputStream();
        outputStream.write("已接收完毕!".getBytes());

        outputStream.close();
        fos.close();
        inputStream.close();
        socket.close();
        serverSocket.close();
    }
}
```

### 4.2 客户端

```java
package pers.wl.netprograming;

import java.io.*;
import java.net.InetAddress;
import java.net.Socket;

public class FileClient {
    public static void main(String[] args) throws Exception {
        Socket socket = new Socket(InetAddress.getByName("127.0.0.1"), 9000);
        OutputStream outputStream = socket.getOutputStream();

        FileInputStream fis = new FileInputStream(new File("send.png"));
        byte[] buffer = new byte[1024];
        int len = 0;

        while ((len = fis.read(buffer)) != -1) {
            outputStream.write(buffer, 0, len);
        }

        // 通知服务器，发送结束
        socket.shutdownOutput();
        System.out.println("发送完毕!");

        InputStream inputStream = socket.getInputStream();
        ByteArrayOutputStream baos = new ByteArrayOutputStream();

        while ((len = inputStream.read(buffer)) != -1) {
            baos.write(buffer, 0, len);
        }

        System.out.println(baos.toString());

        baos.close();
        inputStream.close();
        fis.close();
        outputStream.close();
        socket.close();
    }
}
```

## 5. UDP数据传输

### 5.1 发送端

由于使用字节流，因此长度应使用 `msg.getBytes().length`

```java
package pers.wl.netprograming;

import java.net.*;

public class UDPClient {
    public static void main(String[] args) throws Exception {
        DatagramSocket socket = new DatagramSocket();

        String msg = "测试UDP!";
        InetAddress serverIP = InetAddress.getByName("localhost");
        int port = 9000;
        
        // 由于使用字节流，因此长度应使用 msg.getBytes().length
        DatagramPacket packet = new DatagramPacket(msg.getBytes(), 0, msg.getBytes().length, serverIP, port);

        socket.send(packet);
        socket.close();
    }
}
```



### 5.2 接收端

```java
package pers.wl.netprograming;

import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.SocketException;

public class UDPServer {
    public static void main(String[] args) throws Exception {
        DatagramSocket socket = new DatagramSocket(9000);

        byte[] buffer = new byte[1024];
        DatagramPacket packet = new DatagramPacket(buffer, 0, buffer.length);

        socket.receive(packet);
        System.out.println(packet.getAddress().getHostAddress());
        System.out.println(new String(packet.getData(), 0, packet.getLength()));
        socket.close();
    }
}
```


