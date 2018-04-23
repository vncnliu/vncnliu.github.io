---
title: 造轮子之HttpServer（一）
date: 2016-08-12 00:25:17
category:
- 造轮子
tag:
- http
- socket
---

这里的HttpServer就是一个解析http协议的程序，大家都知道http协议是基于tcp/ip协议的。tcp/ip算是比较底层的了，操作系统基本都通过socket对其进行管理，用java很容易就能写一个简单的server
```java
package cc.vicp.vncnliu.attackServer;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * 简单的http解析器
 * created on 2016/8/10 15:11.
 *
 * @author vncnliu
 * @version default 1.0.0
 */
public class HttpServer {

    public static void main(String[] args) {
        try {
            //绑定端口
            ServerSocket serverSocket  = new ServerSocket(8080);
            while (true){
                Socket socket = serverSocket.accept();
                new Thread(new MessageHandle(socket)).start();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    static class MessageHandle implements Runnable {

        Socket socket;
        MessageHandle(Socket socket){
            this.socket = socket;
        }

        public void run() {
            StringBuilder echo = new StringBuilder();
            BufferedReader in = null;
            PrintWriter out = null;
            try {
                in = new BufferedReader(new InputStreamReader(this.socket.getInputStream()));
                out = new PrintWriter(this.socket.getOutputStream(),true);
                String body = null;
                while(true) {
                    body = in.readLine();
                    if(body==null){
                        break;
                    }
                    System.out.println(body);
                }
                echo.append("HTTP/1.1 200 \r\n");
                echo.append("Content-Type: text/html\r\n");
                echo.append("Content-Length: 109\r\n");
                echo.append("\r\n");
                echo.append("<html>");
                echo.append("<head>");
                echo.append("<title>hello</title>");
                echo.append("</head>");
                echo.append("<body>");
                echo.append("<center>");
                echo.append("<b1>Welcome to my home page.</b1>");
                echo.append("</center>");
                echo.append("</body>");
                echo.append("</html>");
                out.println(echo.toString());
            } catch (Exception e) {
                e.printStackTrace();
            }
            finally {
                System.out.println("链接关闭");
                if(in!=null){
                    try {
                        in.close();
                    } catch (Exception e2) {
                        e2.printStackTrace();
                    }
                }
                if(out!=null){
                    out.close();
                }
                if(this.socket!=null){
                    try {
                        this.socket.close();
                    } catch (IOException e1) {
                        e1.printStackTrace();
                    }
                    this.socket=null;
                }
            }
        }
    }
}
```
代码很简单,但是跑起来就不按我想的来了。浏览器输入http://localhost:8080/结果圈圈一直转个不停
{% asset_img 1.png 浏览器访问结果 %}
程序控制台输出如下
```log
GET / HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.103 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Encoding: gzip, deflate, sdch
Accept-Language: zh-CN,zh;q=0.8
```
可以看到没有走到finally里，那到底哪里阻塞住了呢？输出程序的threads dump
{% asset_img 1.png threads dump %}
>有兴趣的可以看一下idea的threads dump 的文档在[这里](https://www.jetbrains.com/help/idea/2016.2/debug-tool-window-dump.html)

可以看到第二个线程是在等待网络数据，这里就要牵扯到http协议了，一个http请求使用tcp/ip协议建立一个socket链接等待response返回数据后关闭链接，InputStream关闭后系统会发送EOF这是read就会返回-1表明流读完了，而请求没有收到返回必然不会关闭链接所以这里会阻塞在socket的inputStream的fill方法。
这里需要结合http协议改变一下代码
```java
package cc.vicp.vncnliu.attackServer;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * 简单的http解析器
 * created on 2016/8/10 15:11.
 *
 * @author vncnliu
 * @version default 1.0.0
 */
public class HttpServer {

    public static void main(String[] args) {
        try {
            //绑定端口
            ServerSocket serverSocket  = new ServerSocket(8080);
            while (true){
                Socket socket = serverSocket.accept();
                new Thread(new MessageHandle(socket)).start();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    static class MessageHandle implements Runnable {

        Socket socket;
        MessageHandle(Socket socket){
            this.socket = socket;
        }

        public void run() {
            StringBuilder echo = new StringBuilder();
            BufferedReader in = null;
            PrintWriter out = null;
            int contentSize = 0;
            try {
                in = new BufferedReader(new InputStreamReader(this.socket.getInputStream()));
                out = new PrintWriter(this.socket.getOutputStream(),true);
                String body = null;
                while(true) {
                    body = in.readLine();
                    if(body==null){
                        break;
                    }
                    if("".equals(body)){
                        //开始读取主题部分，Content-Length为主体大小，
                        char[] content = new char[contentSize];
                        in.read(content);
                        System.out.println(content);
                        break;
                    }
                    if("Content-Length".equals(body.split(":")[0])){
                        contentSize = Integer.parseInt(body.split(":")[1].trim());
                    }
                    System.out.println(body);
                }
                echo.append("HTTP/1.1 200 \r\n");
                echo.append("Content-Type: text/html\r\n");
                echo.append("Content-Length: 109\r\n");
                echo.append("\r\n");
                echo.append("<html>");
                echo.append("<head>");
                echo.append("<title>hello</title>");
                echo.append("</head>");
                echo.append("<body>");
                echo.append("<center>");
                echo.append("<b1>Welcome to my home page.</b1>");
                echo.append("</center>");
                echo.append("</body>");
                echo.append("</html>");
                out.println(echo.toString());
            } catch (Exception e) {
                e.printStackTrace();
            }
            finally {
                System.out.println("链接关闭");
                if(in!=null){
                    try {
                        in.close();
                    } catch (Exception e2) {
                        e2.printStackTrace();
                    }
                }
                if(out!=null){
                    out.close();
                }
                if(this.socket!=null){
                    try {
                        this.socket.close();
                    } catch (IOException e1) {
                        e1.printStackTrace();
                    }
                    this.socket=null;
                }
            }
        }
    }
}
```
参考http权威指南里的说法
>HTTP 报文是简单的格式化数据块。每条报文都包含一条来自客户端的请求，或者一条来自服务器的响应。它们由三个部分组成：对报文进行描述的起始行（start line）、包含属性的首部（header）块，以及可选的、包含数据的主体（body）部分。
 起始行和首部就是由行分隔的 ASCII 文本。每行都以一个由两个字符组成的行终止序列作为结束，其中包括一个回车符（ASCII 码 13）和一个换行符（ASCII 码 10）。这个行终止序列可以写做 CRLF。需要指出的是，尽管 HTTP 规范中说明应该用 CRLF 来表示行终止，但稳健的应用程序也应该接受单个换行符作为行的终止。有些老的，或不完整的 HTTP 应用程序并不总是既发送回车符，又发送换行符。 
 实体的主体或报文的主体（或者就称为主体）是一个可选的数据块。与起始行和首部不同的是，主体中可以包含文本或二进制数据，也可以为空。 
 Content-Type 行说明了主体是什么——在这个例子中，就是纯文本文档。Content-Length 行说明了主体有多大
 
 所以只要把Content-Length读取完就可以发送response了
 >如果为get方法是没有Content-Length也可以认为Content-Length为0
 {% asset_img 3.png 浏览器访问结果 %}