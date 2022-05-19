# Http

HTTP协议是Hyper Text Transfer Protocol（超文本传输协议）的缩写,是用于从万维网（WWW:World Wide Web ）服务器传输超文本到本地浏览器的传送协议。  
HTTP是一个基于`TCP/IP`通信协议来传递数据（HTML 文件, 图片文件, 查询结果等）。  

## 请求报文

---

### 客户端请求消息

客户端发送一个HTTP请求到服务器的请求消息包括以下格式：请求行（request line）、请求头部（header）、空行和请求数据四个部分组成，下图给出了请求报文的一般格式。

![请求报文](pic/2012072810301161.png)

下面实例是一点典型的使用GET来传递数据的实例：  
客户端请求：  

    行  POST /hello.txt HTTP/1.1
    头  User-Agent: curl/7.16.3 libcurl/7.16.3 OpenSSL/0.9.7l zlib/1.2.3
        Host: www.example.com
        Accept-Language: en, mi
    空行
    体  username=admin&password=admin

### 服务器响应消息

HTTP响应也由四个部分组成，分别是：状态行、消息报头、空行和响应正文。  

![响应报文](pic/httpmessage.jpg)

服务端响应如下

    HTTP/1.1 200 OK
    Date: Mon, 27 Jul 2009 12:28:53 GMT
    Server: Apache
    Last-Modified: Wed, 22 Jul 2009 19:15:56 GMT
    ETag: "34aa387-d-1568eb00"
    Accept-Ranges: bytes
    Content-Length: 51
    Vary: Accept-Encoding
    Content-Type: text/plain