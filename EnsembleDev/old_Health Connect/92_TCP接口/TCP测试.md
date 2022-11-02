# TCP预置组件测试



## Test 1. EnsLib.HL7.Service.TCPService -> EnsLib.HL7.Operation.TCPOperation

  description: BS使用EnsLib.HL7.Adapter.TCPInboundAdapter，消息使用Ens.Hl7.Message
  result: pass

## Test2. EnsLib.TCP.StatusService 

```
CNMBPHMA:Tester hma$ nc 172.16.58.200 9001
namespace
DEMO
production
SEDemo.IO.TCPProduction
utctime
2021-04-16 07:28:04.929
version
2021.1.0.187
quit
CNMBPHMA:Tester hma$
```

> EnsLib.XML.TCPService extends EnsLib.TCP.PassthroughService, 里面也没实现说明东西， 这个有什么用？ 太奇怪了

## Test4. EnsLib.TCP.PassthroughService

   - 使用的是EnsLib.TCP.CountedInboundAdapter
   - 配置项：
     - SendAcknowledgement: 是否让client知道收到了请求，默认选中
     - GetStreamName: 是否期望收到一个"stream name text string prefix"before收到每个stream.默认选中， 但是为什么？
     - 使用FileStream,如果不选中， 使用的是globalstream, 
   - 用的message是"Ens.StreamContainer"
   - 测试过程：
     - 如果用FileStream, 产生的消息中Type是FC, 
     - 如果不用FileStream, 消息是这样，也还是没有stream内容。Ens.StreamContainer里面有StreamGC, 但是内部的proprety, Stream是calculated, 应该是可以用的呀？

```
<StreamContainer xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:s="http://www.w3.org/2001/XMLSchema">
<OriginalFilename>TCP2634</OriginalFilename>
<Stream></Stream>
<Type>GC</Type>
</StreamContainer>
```



## Test 5: TextLine BO

创建一个使用TextLinOutbound Adapter的BO, 用组件测试发送ping, 结果没问题， 保存May8_3.pcap
和上一个测试同样使用的是"保持连接=0”，但这次的SYN对方Redis是认识的， 为什么？

    Method process(pRequest As Ens.StringContainer, Output pResponse As Ens.StringContainer) As %Status
    {
      set pResponse=##class(Ens.StringContainer).%New()
      set tsc=..Adapter.SendMessageString(pRequest.StringValue,.a)
      set pResponse.StringValue=a
      Quit tsc
    }

**问题： 这个Adapter没有SendMessageStream(),真是神奇




## Test 6. EnsLib.TCP.FramedPassthroughService/Operation

发送消息自Mac Netcat： 
      nc 172.16.58.200 9004

**Service配置**

- 放弃不正确的结果， 不选
- 删除结构， 不选
- 保持连接： 0 
- 消息贞开始=11， 消息贞结束 = 28， 13（默认)
- Ack ok=6, Ack not ok=21 (默认)
- 消消是StreamContainer, Type=CG，中文也没问题， 

**Operation配置** 

- 保持连接： 0 
- 消息贞开始=11， 消息贞结束 = 28， 13（默认)

![](/Users/hma/MySE/hma培训教材/HealthConnect基础/TCP接口/InterSystemsNotebook/pictures/TCPFramedPassthrough.png)

**测试结果, May8_4.pcap**

> BS可以把消息发送给BO, 但BO收到后没有接到从Redis回来的"PONG",查看抓包发现，May8_1.pcap里面，TCPFramedPassthrough BO向Redis发送了一个SYN,收到一个RST，不知道什么原因。和以前的有关系？ 重启Redis和BO, 再测，结果一样。May8_2.pcap（answer: 发送的接口错了)

发送的psh分成了3个， 一个"ob"字节，一个"ping.", 一个"1c0d0a",好像是我设置的framed标识符

Redis返回了“PONG -ERR unknown n command '.', with args beginning with:.."

去掉再试： 
**May8_5.pcap**
BO发了一个请求， redis返回了+PONG，唯一的问题是message viewer上没有出错，但返回消息里只有一个+。最终呼叫方netcat也是收到一个+

    <StreamContainer xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:s="http://www.w3.org/2001/XMLSchema">
    <Stream>+</Stream>
    <Type>GC</Type>
    </StreamContainer>

换别的客户端试试：



## Test TCP and UDP with netcat(nc)

https://www.digitalocean.com/community/tutorials/how-to-use-netcat-to-establish-and-test-tcp-and-udp-connections-on-a-vps

 

 

CNMBPHMA:~ hma$ nc -z -v localhost 20-30

nc: connectx to localhost port 20 (tcp) failed: Connection refused

nc: connectx to localhost port 20 (tcp) failed: Connection refused

nc: connectx to localhost port 21 (tcp) failed: Connection refused

nc: connectx to localhost port 21 (tcp) failed: Connection refused

found 0 associations

found 1 connections:

   1:    flags=82<CONNECTED,PREFERRED>

outif lo0

src ::1 port 57614

dst ::1 port 22

rank info not available

TCP aux info available

 

Connection to localhost port 22 [tcp/ssh] succeeded!

nc: connectx to localhost port 23 (tcp) failed: Connection refused

nc: connectx to localhost port 23 (tcp) failed: Connection refused

nc: connectx to localhost port 24 (tcp) failed: Connection refused

nc: connectx to localhost port 24 (tcp) failed: Connection refused

nc: connectx to localhost port 25 (tcp) failed: Connection refused

nc: connectx to localhost port 25 (tcp) failed: Connection refused

nc: connectx to localhost port 26 (tcp) failed: Connection refused

nc: connectx to localhost port 26 (tcp) failed: Connection refused

nc: connectx to localhost port 27 (tcp) failed: Connection refused

nc: connectx to localhost port 27 (tcp) failed: Connection refused

nc: connectx to localhost port 28 (tcp) failed: Connection refused

nc: connectx to localhost port 28 (tcp) failed: Connection refused

nc: connectx to localhost port 29 (tcp) failed: Connection refused

nc: connectx to localhost port 29 (tcp) failed: Connection refused

nc: connectx to localhost port 30 (tcp) failed: Connection refused

nc: connectx to localhost port 30 (tcp) failed: Connection refused

CNMBPHMA:~ hma$

 

//对端开防火墙， 没有反应

CNMBPHMA:~ hma$ nc -z -v 172.19.86.91 9000-9010

^C

CNMBPHMA:~ hma$ 

//本地VM的port

CNMBPHMA:~ hma$nc -z -v 172.16.58.200 9000-9010

nc: connectx to 172.16.58.200 port 9000 (tcp) failed: Connection refused

nc: connectx to 172.16.58.200 port 9001 (tcp) failed: Connection refused

nc: connectx to 172.16.58.200 port 9002 (tcp) failed: Connection refused

nc: connectx to 172.16.58.200 port 9003 (tcp) failed: Connection refused

nc: connectx to 172.16.58.200 port 9004 (tcp) failed: Connection refused

nc: connectx to 172.16.58.200 port 9005 (tcp) failed: Connection refused

nc: connectx to 172.16.58.200 port 9006 (tcp) failed: Connection refused

nc: connectx to 172.16.58.200 port 9007 (tcp) failed: Connection refused

nc: connectx to 172.16.58.200 port 9008 (tcp) failed: Connection refused

nc: connectx to 172.16.58.200 port 9009 (tcp) failed: Connection refused

nc: connectx to 172.16.58.200 port 9010 (tcp) failed: Connection refused

CNMBPHMA:~ hma$

 

如果不使用DNS，可以用'-n'加快速度

CNMBPHMA:~ hma$ nc -z -v -n 172.16.58.200 9000-9010

nc: connectx to 172.16.58.200 port 9000 (tcp) failed: Connection refused

nc: connectx to 172.16.58.200 port 9001 (tcp) failed: Connection refused

nc: connectx to 172.16.58.200 port 9002 (tcp) failed: Connection refused

nc: connectx to 172.16.58.200 port 9003 (tcp) failed: Connection refused

nc: connectx to 172.16.58.200 port 9004 (tcp) failed: Connection refused

nc: connectx to 172.16.58.200 port 9005 (tcp) failed: Connection refused

nc: connectx to 172.16.58.200 port 9006 (tcp) failed: Connection refused

nc: connectx to 172.16.58.200 port 9007 (tcp) failed: Connection refused

nc: connectx to 172.16.58.200 port 9008 (tcp) failed: Connection refused

nc: connectx to 172.16.58.200 port 9009 (tcp) failed: Connection refused

nc: connectx to 172.16.58.200 port 9010 (tcp) failed: Connection refused

CNMBPHMA:~ hma$

 

 

//关掉目标的防火墙后，保存wireshark文件

CNMBPHMA:~ hma$ nc -z -v 172.19.86.91 9000-9010

nc: connectx to 172.19.86.91 port 9000 (tcp) failed: Connection refused

found 0 associations

found 1 connections:

   1:    flags=82<CONNECTED,PREFERRED>

outif en0

src 172.19.86.90 port 57881

dst 172.19.86.91 port 9001

rank info not available

TCP aux info available

 

Connection to 172.19.86.91 port 9001 [tcp/etlservicemgr] succeeded!

nc: connectx to 172.19.86.91 port 9002 (tcp) failed: Connection refused

nc: connectx to 172.19.86.91 port 9003 (tcp) failed: Connection refused

nc: connectx to 172.19.86.91 port 9004 (tcp) failed: Connection refused

nc: connectx to 172.19.86.91 port 9005 (tcp) failed: Connection refused

nc: connectx to 172.19.86.91 port 9006 (tcp) failed: Connection refused

nc: connectx to 172.19.86.91 port 9007 (tcp) failed: Connection refused

nc: connectx to 172.19.86.91 port 9008 (tcp) failed: Connection refused

nc: connectx to 172.19.86.91 port 9009 (tcp) failed: Connection refused

nc: connectx to 172.19.86.91 port 9010 (tcp) failed: Connection refused

CNMBPHMA:~ hma$

 

acting as a TCP server. Netcat can listen for the specified TCP port. But as a security measure in Linux systems only privileged users can listen to ports between 1-1024 . In this example, we will listen to TCP ports 30. To give required privileges we use sudo command.

 

 

CNMBPHMA:~ hma$ nc -l 4444

 

hello

world

CNMBPHMA:~ hma$

CNMBPHMA:~ hma$

 

 

TCP Client

 

//connect to Dell HealthConnect, "EnsLib.TCP.StatusService"

CNMBPHMA:~ hma$ nc 172.19.86.91 9002

namespace

HMA

production

SEDemo.WS.XSLTDemo.Production

utcstarttime

2019-12-23 01:39:48.877

utctime

2019-12-23 02:53:49.623

version

2018.1.0.184

quit

CNMBPHMA:~ hma$

 

奇怪的是断掉了对端不告警， 虽然设置的也是调用间隔是5s, 但表现和EnsLib.HL7.Service.TCPService不一样， 那个连接断了就变红

 

 

 

//连接对端EnsLib.HL7.Service.TCPService不成功， CNMBPHMA:~ hma$ nc 172.19.86.91 9001

MSH|^~\&|REGADT|MCM|IFENG||199601061253||ADT^A01|000001|P|2.3.1||||||utf-8

MSH|^~\&|REGADT|MCM|IFENG||199601061253||ADT^A01|000001|P|2.3.1||||||utf-8

^[[A^[[B

MSH|^~\&|

 

^C

CNMBPHMA:~ hma$

 

 

对方收到消息，但Terminal显示错误信息:

 

...TCPInboundAdapter: ERROR<Ens>ErrTCPTerminatedReadTimeOutExpired: TCP Rea timeout(5) expired waiting for terminator EndData=13, data recevied ='MSH|^~\&|REGADT|MCM|IFENG||199601061253||ADT^A01|000001|P|2.3.1||||||utf-8                                         '

 

似乎是因为发送的包的行结尾，要不就是文件结尾不对， 所以， 用文件试试

 

//Perfect: 对端收到

 

CNMBPHMA:Desktop hma$ ls *.txt

ADT_A01.txt

CNMBPHMA:Desktop hma$ nc 172.19.86.91 9001 <ADT_A01.txt

CNMBPHMA:Desktop hma$ SC|REGADT|MCM|201912231129||ACK^A01|000001|P|2.3.1

CNMBPHMA:Desktop hma$

 

 

as a Simple Web Server

 

//index.html

<html>

​    <head>

​        <title>Test Page</title>

​    </head>

​    <body>

​        <h1>Level 1 header</h1>

​        <h2>Subheading</h2>

                <p>Normal text here</p>

​    </body>

</html>

 

//Run 

 

CNMBPHMA:Desktop hma$ printf 'HTTP/1.1 200 OK\n\n%s' "$(cat index.html)" | nc -l 8888

GET / HTTP/1.1

Host: localhost:8888

Connection: keep-alive

Cache-Control: max-age=0

DNT: 1

Upgrade-Insecure-Requests: 1

User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36

Sec-Fetch-User: ?1

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9

Sec-Fetch-Site: none

Sec-Fetch-Mode: navigate

Accept-Encoding: gzip, deflate, br

Accept-Language: zh-CN,zh;q=0.9,en;q=0.8

Cookie: Username=_SYSTEM; state-35A8B702-F451-11E9-AF34-0242C0A80102=SYSADM%3A0%2C0; CacheBrowserId=OQl66OPj7fWq6yXmkRMj9A; CSPWSERVERID=hA00Dipn; state-13BDDCBC-1B20-11EA-80C0-0242AC110002=ENS%3A0%2C0

 

CNMBPHMA:Desktop hma$

 

//最后

 

http://localhost:8888/

可以显示上述HTML页面， 但只是一次， 连接后就nc的执行就结束了

 

 

//以下命令可以无限循环， 直到用ctrl+C

 

while true; do printf 'HTTP/1.1 200 OK\n\n%s' "$(cat index.html)" | netcat -l 8888; done

 

问题？ 有没有更简单的办法

