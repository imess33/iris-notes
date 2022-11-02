# Test 1



TextLine BO: 

- Stay Connect = -1
- connect Timeout= 5
- Reconnect Retry = 5
- Get Reply = yes
- ResponseTimeout=15

测试结果： 当sever断了， 从BO发送消息15秒超时，消息状态是error; job状态是error，host变红， 恢复server连接后可以继续发消息， 然后host变绿， job状态也是ok了， 

<Ens>ErrOutConnectExpired: 172.16.58.1:8888 的 TCP 连接超时期 (5) 已过

应该是连接就超时，所以发送数据没有包就没有发出去，这样不会有重发的问题。 

打开archive IO, 发现BO向adapter发了3次

10s收到请求，然后立刻发了一次， 20s发了一次， 25s发了第3次， 然后就error了。 

