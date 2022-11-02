# 开发Ensemble REST服务

> REF: https://docs.intersystems.com/healthconnectlatest/csp/docbook/Doc.View.cls?KEY=GREST
>
> REF: https://docs.intersystems.com/healthconnectlatest/csp/docbook/DocBook.UI.Page.cls?KEY=AFL_rest#AFL_C4838



新手学习在Health Connect中开发REST服务时会感觉非常混乱。早期的Health Connect版本中很多实现不成熟是最大的原因， 还有就是文档结构不清晰。 

好消息是最新的2021版本的Health Connect中， 无论是实现方法还是文档， 都有非常大的进步，基于最新的这个版本，下面是我整理了一下开发REST接口的大概过程， 起个入门的作用。 

在介绍具体步骤之前，先明确两个要点：

1. **REST服务和Ensemble是完全单独的两个东西。**

做Health Connect开发就是要开发Ensemble Production, 使用BS， BP， BO来实现消息的传输，也就是实现一个messaging system。 其他的接口有预装的组件，使用相应的适配器，比如TCPyouTCP的， SOAP有SOAP的，但REST服务没有预装组件， 也没有适配器。 

> 如果您看的了代码包里的EnsLib.REST.Service类， 它继承了%CSP.REST页面， 也继承了BusinessService，非常符合Ensemble的结构设计。But, 别用。在线文档中有专门的说明。" Although InterSystems IRIS defines a class EnsLib.REST.ServiceOpens in a new window, that is a subclass of %CSP.RESTOpens in a new window, we recommend that you not use this class because it provides an incomplete implementation of %CSP.REST Opens in a new window."

从理论上说， REST服务和Ensemble也是两回事。REST是一个Web服务， 开发这个服务使用的技术叫“CSP”（Cache‘ Server Page）， 你可以把他理解为JSP， ASP同类的东西。Web Application收到一个REST请求，返回响应消息， 这个实现的类叫%CSP.REST. 本身是和Ensemble没关系的。

2. **Health Connect提供REST服务的场景**
3. 如果是Health Connect本身提供服务，就根本没必要创建Ensemble Production, 使用Ensemble Message. 如果是要把一个

1. 









开发REST服务有两个方式， 一个是生生的写代码， 定义接口的标准，被称为"Manually Coding"。第2个方式是目前越来越流行的"Sepcification-first"，也就是使用描述性的语言定义接口规范，然后通过这个规范生成接口代码。第2种方式更快捷，但这里我还是从第一种介绍起，对理解里面的代码层次更容易一些，而这是调试一个接口必须的。 

## 从代码开发REST服务

不同于HTTP和SOAP, Ensemble里面没有REST的inbound Adaptor，也没有可用的BS组件。在Production里开发一个REST服务的步骤是：

1. 开发一个REST Service, 这个Service是一个CSP Page, 是一个网页服务，和Ensemble没关系。要在Production中使用这个服务，您需要在这个服务里调用一个Production的业务服务BS。
2. 要访问这个REST页面服务， 您需要配置一个Web Application。Web Application的配置项上有一个选项: "REST 分派类"。这样配置好之后， Web Application收到相应的URL后就会调用这个REST页面，页面再去调用Production的BS。
3. 最后，您需要在BS中处理收到的JSON, 发送给其他组件，以传递给接收方系统。

> 

让我们开始开发一个简单的REST服务并加入Production:

### Step 1:

创建以下代码，解释一下：

- 继承%CSP.REST，这是个专用于REST的CSP页面
- UrlMap是一个XData, 在COS语言里用于在代码里放置固定的xml数据结构。UrlMap定义从收到的URL到本类里不同的方法之间的映射。
- 方法中入参可以是任意的数据结构和用户定义的类结构，不需要出参。如果直接返回消息给调用者，直接"write"一个流或者字符串

```
Class SEDemo.IO.REST.SampleService Extends %CSP.REST
{

  XData UrlMap [ XMLNamespace = "http://www.intersystems.com/urlmap" ]
  {	<Routes>
          <Route Url="/Patient/:Id" Method="GET" Call="GetPatientById"/>
          <Route Url="/Test/:id" Method="GET" Call="Test"/>
     </Routes>
  }

  ClassMethod Test(pInput As %String) As %Status
  {
  	write "Received: "_pInput，！
    Quit 1
  }

  ClassMethod GetPatientById(pID As %String) As %Status
  { 	Try{
      	Set tObj=##class(SEDemo.Common.Patient).%OpenId(pID),tStream = ""
      	d ##class(%ZEN.Auxiliary.jsonProvider).%WriteJSONStreamFromObject(.tStream,tObj)
      	w tStream.Read()
      } Catch (e) {Set tSC=e.AsStatus()}
      Quit tSC
  }
}
```

### Step 2: 创建Web Application

在管理界面System Administration > Security > Applications > Web Applications，创建一个用于接收此REST服务的Web APPlication, 设置"Dispatch Class"为当前类。 假设创建的Web Applicaiton为"/CSP/myrest",注意：

- 选中“Enable Application"
- 权限: 分配本命名空间数据库的资源，默认是%DB_%Default。

*后面会详细介绍权限和用户管理的细节。*

### Step 3: 测试你的REST service

你可以选择自己喜欢的测试方式，比如用浏览器，POSTMAN, SoapUI...， 下面是我测试的记录：

 ```
 CNMBPHMA:~ hma$ curl -v http://172.16.58.200:52773/csp/myrest/Test/333
 *   Trying 172.16.58.200...
 * TCP_NODELAY set
 * Connected to 172.16.58.200 (172.16.58.200) port 52773 (#0)
 > GET /csp/myrest/Test/333 HTTP/1.1
 > Host: 172.16.58.200:52773
 > User-Agent: curl/7.54.0
 > Accept: */*
 >
 < HTTP/1.1 200 OK
 < Date: Wed, 14 Jul 2021 06:47:26 GMT
 < Server: Apache
 < CACHE-CONTROL: no-cache
 < EXPIRES: Thu, 29 Oct 1998 17:04:19 GMT
 < PRAGMA: no-cache
 < CONTENT-LENGTH: 15
 < Content-Type: text/html; charset=utf-8
 <
 Received: 333
 * Connection #0 to host 172.16.58.200 left intact
 CNMBPHMA:~ hma$
 ```

这里是一个匿名访问，如果需要用户认证，修改一下重发：

```
CNMBPHMA:~ hma$ curl -u 'superuser:SYS' http://172.16.58.200:52773/csp/myrest/Test/333
Received: 333
CNMBPHMA:~ hma$
```

注意两点：

1. 到目前为止我们测试的其实是一个HTTP请求和响应，虽然内部用了%CSP.REST的类， 但响应中'Content-Type'还是'text/html'
2. 代码中没有处理出错和查不到结果的情况
3. 到目前为止和Ensemble Production没有任何关系。



## Step 4: 将服务加入Ensemble Production

加入Production的意思实际上时调用一个Production的BusinessService。

让我们先创建一个简单的Service.

```
///不使用Adapter, 收到任何请求，同步发送给目标组件

Class Test.BS.GeneralService Extends Ens.BusinessService
{
  Method OnProcessInput(pInput As %RegisteredObject, Output pOutput As %RegisteredObject) As %Status
  {
    set tRequest=##class(Ens.StringRequest).%New()
    Set tStatus = ..SendRequestSync("Test.BO.dummyOperation", tRequest, .tResponse)
    set pOutput = tResponse
    Quit tStatus
  }
}
```

当需要前面的REST服务来调用这个BusinessService的时候， 需要在method里面加入直接调用的语句，比如上面的GetPatientById()

```
ClassMethod GetPatientById(pID As %String) As %Status
{ 	
  set status = ##class(Ens.Director).CreateBusinessService("Test.BS.GeneralService", .tService)
  if $$$ISOK(status) {
	   set status = service.OnProcessInput(pID, .tResponse)
  }
  w tResponse,!
}
```







## Advanced Skills

expire, content-length, keep-alive, connection, content-type







# End









### 更多的例子

[First Look: Developing REST Interfaces in InterSystems Products](https://docs.intersystems.com/iris20181/csp/docbook/DocBook.UI.Page.cls?KEY=AFL_rest)

[Code Sample](https://github.com/intersystems/FirstLook-REST)

 sample from ensdemo: Demo.Rest Package

 ### 免费的API接口

https://www.zhihu.com/question/32225726

https://api.gugudata.com/weather/weatherinfo/demo

https://dog.ceo/api/breeds/image/random

https://postman-echo.com/get?foo1=bar1&foo2=bar2

https://docs.postman-echo.com/?version=latest

### 我自己的Postman API key

 PMAK-5e8c3c706d31560043db278f-0e08d5687a3bf72dd19d776a6402631ca6