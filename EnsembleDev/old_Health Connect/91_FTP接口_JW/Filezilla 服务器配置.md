#  # FTP 服务端搭建 

##  1. 下载filezilla server https://filezilla-project.org/

<img src="source/Screen Shot 2021-07-14 at 2.07.57 PM.png" alt="Screen Shot 2021-07-14 at 2.07.57 PM" style="zoom:30%;" />

## 2. add user (用于IRIS的 credentials 凭据配置)

<img src="source/Screen Shot 2021-07-14 at 9.54.52 AM.png" alt="Screen Shot 2021-07-14 at 9.54.52 AM" style="zoom:30%;" />

##  3. 创建共享文件夹

<img src="/Users/jinwang/Documents/学习笔记/河南新星培训/FTP/source/Screen Shot 2021-07-14 at 10.10.41 AM.png" alt="Screen Shot 2021-07-14 at 10.10.41 AM" style="zoom:30%;" />

##  4. 防火墙设置：

在防火墙的inbound rules 里面加一个TCP的规则，开放端口21，20，1234，60000-60020

<img src="/Users/jinwang/Documents/学习笔记/河南新星培训/FTP/source/Screen Shot 2021-07-14 at 10.22.02 AM.png" alt="Screen Shot 2021-07-14 at 10.22.02 AM" style="zoom:30%;" />

<img src="/Users/jinwang/Documents/学习笔记/河南新星培训/FTP/source/Screen Shot 2021-07-14 at 10.23.27 AM.png" alt="Screen Shot 2021-07-14 at 10.23.27 AM" style="zoom:30%;" />

## 5. Pasv模式设置

设置中，把port range 60000 - 60020加上，由于filezilla使用了Passive Mode模式。

<img src="/Users/jinwang/Documents/学习笔记/河南新星培训/FTP/source/Screen Shot 2021-07-14 at 10.24.25 AM.png" alt="Screen Shot 2021-07-14 at 10.24.25 AM" style="zoom:30%;" />

**tips：**

**主动模式和被动模式的的区别：**

先假设客户端为C,服务端为S.

**Port模式:**

当客户端C向服务端S连接后，使用的是Port模式,那么客户端C会发送一条命令告诉服务端S(客户端C在本地打开了一个端口N在等着你进行数据连接),当服务端S收到这个Port命令后 就会向客户端打开的那个端口N进行连接，这种数据连接就生成了。

**Pasv模式:**

当客户端C向服务端S连接后，服务端S会发信息给客户端C,这个信息是(服务端S在本地打开了一个端口M,你现在去连接我吧),当客户端C收到这个信息后，就可以向服务端S的M端口进行连接,连接成功后，数据连接也建立了。

**总结**

从上可知两种模式主要的不同是数据连接建立的不同：

　　对于Port模式，是客户端C在本地打开一个端口等服务端S去连接建立数据连接；

　　而Pasv模式就是服务端S打开一个端口等待客户端C去建立一个数据连接。

## 6. no transfer timeout 设置

<img src="/Users/jinwang/Documents/学习笔记/河南新星培训/FTP/source/Screen Shot 2021-07-14 at 1.43.01 PM.png" alt="Screen Shot 2021-07-14 at 1.43.01 PM" style="zoom:30%;" />

## 7. 测试FileZilla服务器是否可以正常接入

可以找另外一台Windows或者用一台虚拟机，安装FileZilla客户端，用来测试已安装的FileZilla服务器是否可以正常接入。

<img src="/Users/jinwang/Documents/学习笔记/河南新星培训/FTP/source/Screen Shot 2021-07-14 at 2.00.27 PM.png" alt="Screen Shot 2021-07-14 at 2.00.27 PM" style="zoom:30%;" />

## 8. 文件传输成功

可以查看log，检测文档传输是否成功。

<img src="/Users/jinwang/Documents/学习笔记/河南新星培训/FTP/source/Screen Shot 2021-07-14 at 10.44.30 AM.png" alt="Screen Shot 2021-07-14 at 10.44.30 AM" style="zoom:20%;" />

