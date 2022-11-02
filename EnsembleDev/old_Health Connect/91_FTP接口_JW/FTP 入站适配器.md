# IRIS HealthShare FTP 接口示例

本示例是用InterSystems IRIS抓取FTP服务器中的文件，并以字符流的形式传入，然后通过InterSystems HS保存在本地。

<img src="source/Screen Shot 2021-07-15 at 2.51.37 PM.png" alt="Screen Shot 2021-07-15 at 2.51.37 PM" style="zoom:40%;" />

## 1. 创建一个FTP服务器，本示例采用FileZilla Server。

关于如何搭建一个FileZilla服务器，请参考 FileZilla[服务器配置](/Users/jinwang/Documents/学习笔记/河南新星培训/FTP/Filezilla 服务器配置.md)

## 2. 选择适当的入站适配器：

可以根据需求选择适当地入站适配器，本示例选用FTP 入站适配器(EnsLib.FTP.InboundAdapter)。

适配器的作用：

- 帮助InterSystems IRIS通过FTP协议接受文件
- 该适配器从配置的位置接收FTP输入，读取输入，并将输入作为一个流发送到相关的业务服务(BS)。你创建和配置的业务服务使用这个流，可与其他部分进行通信。

- 如果 FTP 服务器期望对其输入的确认或响应，业务服务也负责制定该响应并通过 EnsLib.FTP.InboundAdapter 将其转发回数据源。

整体流程：

<img src="source/Screen Shot 2021-07-14 at 1.48.06 PM.png" alt="Screen Shot 2021-07-14 at 1.48.06 PM" style="zoom:40%;" />

## 3. 创建一个业务服务类:

要在生产中使用入站适配器，需要创建一个新的业务服务类(BS)。然后，把它添加到你的生产中并配置它。

你的业务服务类应该扩展Ens.BusinessService，在你的类中，ADAPTER参数应该等于EnsLib.FTP.InboundAdapter，具体操作如下所示：

- 用Studio建一个业务服务类BS。

<img src="/Users/jinwang/Documents/学习笔记/河南新星培训/FTP/source/Screen Shot 2021-07-14 at 2.55.22 PM.png" alt="Screen Shot 2021-07-14 at 2.55.22 PM" style="zoom:30%;" />

- 选择EnsLib.FTP.InboundAdapter为Associated Inbound Adapter。

<img src="/Users/jinwang/Documents/学习笔记/河南新星培训/FTP/source/Screen Shot 2021-07-14 at 2.56.40 PM.png" alt="Screen Shot 2021-07-14 at 2.56.40 PM" style="zoom:30%;" />

- 新的业务服务类BS显示如下，返回值设置成 $$$OK, 并编译。

  将输入流对象包裹在一个StreamContainer消息对象中并发送。

  ```java
  Class Test.FTP.FTPService2 Extends Ens.BusinessService
  {
  
  Parameter ADAPTER = "EnsLib.FTP.InboundAdapter";
  
  /// 将从FTP服务器中获得的字符流装入流容器，发送给业务操作BO。
  Method OnProcessInput(pInput As %Stream.Object, pOutput As %RegisteredObject) As %Status
  {
  	//$$$LOGINFO("start ...")
     	Set tSC=$$$OK
     	set pInput=##class(Ens.StreamContainer).%New(pInput)
      //$$$LOGINFO("processing ...")
  
      $$$LOGINFO("Sending input Stream ...")
      Set tSC1=..SendRequestSync("FileOperation",pInput, .pOutput)
      Set:$$$ISERR(tSC1) tSC=$$$ADDSC(tSC,tSC1)
  
      Quit tSC
  }
  
  }
  ```





## 4. 在管理门户创建业务服务(BS)。

- 创建一个BS: Service Class 选择步骤3中建好的的BS类，选择立即启用。

<img src="source/Screen Shot 2021-07-14 at 3.35.15 PM.png" alt="Screen Shot 2021-07-14 at 3.35.15 PM" style="zoom:30%;" />

- 新增FTP凭据(用户名和密码与FTP服务端一致)：Interoperability -> 配置 -> 凭据 -> 新建

  <img src="source/Screen Shot 2021-07-14 at 3.13.39 PM.png" alt="Screen Shot 2021-07-14 at 3.13.39 PM" style="zoom:30%;" />

- BS基本信息设置

   - 文件路径: 文件读取路径。
   - 从服务器删除：是否读取文件后，将原文件从原服务器中删除
   - 文件名称: 文件名或者使用通配符
   - 存档了路径：读入文件存档路径
   - FTP服务器：FTP服务器IP地址
   - FTP端口：本示例采用FileZilla， 所以端口是21
   - 凭据：FTP凭据(用户资格)

<img src="source/Screen Shot 2021-07-15 at 3.03.28 PM.png" alt="Screen Shot 2021-07-15 at 3.03.28 PM" style="zoom:30%;" />

​	

## 5.创建一个业务操作类:

要在生产中使用出站适配器，需要创建一个新的业务操作类(BO)。然后，把它添加到你的生产中并配置它。

你的业务服务类应该扩展Ens.BusinessService，在你的类中，ADAPTER参数应该等于EnsLib.FTP.InboundAdapter，具体操作如下所示：

- 用Studio建一个业务服务类BO。

  <img src="/Users/jinwang/Documents/学习笔记/河南新星培训/FTP/source/Screen Shot 2021-07-14 at 3.25.37 PM.png" alt="Screen Shot 2021-07-14 at 3.25.37 PM" style="zoom:40%;" />

- 选择EnsLib.File.OutboundAdapter 为Associated Outbound Adapter。

  <img src="/Users/jinwang/Documents/学习笔记/河南新星培训/FTP/source/Screen Shot 2021-07-15 at 2.40.03 PM.png" alt="Screen Shot 2021-07-15 at 2.40.03 PM" style="zoom:40%;" />

- 新的业务操作类BO显示如下，并编译。

  ```java
  Class Test.FTP.FileOperation1 Extends Ens.BusinessOperation
  {
  
  Parameter ADAPTER = "EnsLib.File.OutboundAdapter";
  Parameter INVOCATION = "Queue";
  
  /// BO获取字符流，保存至本地。
  Method FileSendReply(pRequest As Ens.StreamContainer, Output pResponse As Ens.Response) As %Status
  {
      //从流容器中取出字符流
      set stream = ##class(%Stream.GlobalCharacter).%New()
      set stream = pRequest.StreamGet()
      
     	//将字符流转换写入文档并保存
  		set fileName = stream.Attributes("Filename")
   		set st=..Adapter.PutLine(fileName,stream)
   	
   		set pResponse=##class(Ens.StringResponse).%New()
      set pResponse.StringValue="done"
   		Quit $$$OK
  }
  
  XData MessageMap
  {
  <MapItems>
  	<MapItem MessageType="Ens.StreamContainer"> 
  		<Method>FileSendReply</Method>
  	</MapItem>
  </MapItems>
  }
  }
  ```

  

## 6. 在管理门户创建业务操作(BO)

- 在管理门户中创建一个BO，选择步骤5中建好的的BO类，选择立即启用。

<img src="/Users/jinwang/Documents/学习笔记/河南新星培训/FTP/source/Screen Shot 2021-07-14 at 3.33.07 PM.png" alt="Screen Shot 2021-07-14 at 3.33.07 PM" style="zoom:30%;" />

- BS基本信息设置

  - 文件路径: 文件存储路径。

  <img src="/Users/jinwang/Documents/学习笔记/河南新星培训/FTP/source/Screen Shot 2021-07-15 at 3.00.47 PM.png" alt="Screen Shot 2021-07-15 at 3.00.47 PM" style="zoom:30%;" />

  ##  7. 运行Production，查看文档是否正常从FTP服务器中读取并存入本地

