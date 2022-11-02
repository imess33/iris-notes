# HealthConnect中创建HTTP服务端

这里我说说怎么在HealthConnect上开发HTTP服务。

作为消息引擎，HealthConnect会需要从一个接口接收HTTP请求发送到另一个接口，中间做消息转换，路由等等，目的的接口可能是HTTP，或者SOAP，REST等等。这里只介绍HTTP服务的内容，也就是最简单的两种实现：


## 第一种：实现客户定制的HTTP服务业务服务组件(Business Servie)

创建Business Service类，继承EnsLib.HTTP.Service， 如下面的示例:
```java
Class SEDemo.IO.HTTP.ServiceExample1 Extends EnsLib.HTTP.Service
{
    Parameter ADAPTER;
    Method OnProcessInput(pInput As %Stream.Object, Output pOutput As %Stream.Object) As %Status
    {
        //创建Ensemble消息发送给其他组件
        set pRequest=##class(Ens.StreamContainer).%New()
        Set tSC=pRequest.StreamSet(pInput)
        set tSC= ..SendRequestAsync("Dummy1",pRequest,.pResponse)
        //创建返回Stream,发送给调用方
        set pOutput=##class(%Stream.GlobalCharacter).%New()
        do pOutput.Write("yes, I recieved request")
        Quit tSC
    }
}
```
详细说明： 

**使用CSP机制接收请求，不要用HTTP的Inbound Adpater**

如果您学习过在Ensemble上开发SOAP接口，一定对代码里的***"Parameter ADAPTER;"***不陌生，它的作用是确定不要使用父类里的Adapter。

EnsLib.HTTP.Service有两个工种模式，一个是使用内置适配器是Ens.HTTP.InboundAdapter，还有一个是使用CSP机制，从CSP Gateway接收请求。下面的图中黄色的箭头是用适配器接收消息，这时业务服务可以定义工种的URL，端口，SSL等等；下面红色的箭头是所谓的”CSP请求“，也就是HTTP请求经过Web服务器，CSP Gateway， 到Web Application, 再被EnsLib.HTTP.Service收到。

使用CSP机制有更好的安全性和性能，所以在HealthConnect中任何HTTP的服务端接口我们都推荐CSP机制，包括HTTP接口，SOAP接口， REST接口。这些接口的开发都不要使用对应的InboundAdapter。

有关CSP Gateway的原理，还可以参见在线文档或者我的另一技术文章:（到community该文章的链接).


注意的是：当不使用Adapter时，Production页面的组件配置中很多项目会消失，这些是Adapter的属性，比如允许的IP, 端口，包括编码等等。因为不用Adapter，您也不用定义IP,端口号；只有编码，可以在BS的代码里实现。实现的操作可以参考Adapter的设置： https://docs.intersystems.com/healthconnectlatest/csp/docbook/Doc.View.cls?KEY=EHTTP_settings_inbound 

**OnProcessInput()的入参pInput**

业务组件收到的HTTP请求由pInput传入，真正的类型是%CSP.GlobalBinaryStream，它的父类是已经不推荐使用的流类型%GlobalBinaryStream。用%Stream.Object作为pInput的对象类型是合适的，这是一个新版本的Stream对象的超类，可以是任意类型的流。上面代码里业务服务组件发出的Ensemble消息的类型是StreamContainer，如果你看看消息跟踪的类型，你会发现里面流的类型是”GB",也就是一个%GlobalBinaryStream类型的流。

pInput对象的属性Stream里存放的是HTTP Body，而HTTP头放在Attributes属性里，如下图所示：
![pInputStream](pictures/ehttp_request_to_stream.png)  

**如何获得请求里的消息头**  

以下是用pInput.GetAttributeList()得到的Attributes的内容：
        <![CDATA[*CSPApplication/csp/healthshare/demo/CharEncoding1EnsConfigName SEDemo.IO.HTTP.ServiceExample1
        HTTPVersion1.1
        HttpRequestGET
            IParamsParamsRawParamsTranslationTableRAWAURL:/csp/healthshare/demo/SEDemo.IO.HTTP.ServiceExample1.clsaccept*/*"accept-encodinggzip, deflatecache-control
        no-cacheconnectionkeep-alivecontent-length0content-type£cookieCSPSESSIONID-SP-80-UP-csp-healthshare-=0000010100002ROlNxwgxwUQuPHKQogHGNWfADLfF2xiPwce2s; CacheBrowserId=ui$4lIZ_rJsPD_xUTPG$Rw; CSPWSERVERID=B33PnJAehost172.16.58.200mykeyimess7postman-token&83e942ce-ae73-4d98-b7ab-1e2803c95794%user-agentPostmanRuntime/7.18.0]]>

而获取消息头的某些值， 可以用 pInput.Attributes(http_header_name)得到， 比如下面这些：

    set MethodTypeName=pInput.Attributes("HttpRequest")
    set HTTPVersion = pInput.Attributes("HTTPVersion")
    set ContentType = pInput.Attributes("content-type")
    set ContentLength = pInput.Attributes("content-length")
    set URL = pInput.Attributes("URL")
    //获得GET请求，得到URL中的form参数（注意，POST请求的表单数据在消息体里面)
    set pList = Attributes("Params",form_variable_name,n), 
    //得到自定义的消息头值，比如上面的消息里面有个自定义的头字段“mykey"
    set Mykey = pInput.Attributes("mykey")

更多的细节，请参考在线文档: [Using the HTTP Inbound Adap: About the Attributes Array](https://docs.intersystems.com/healthconnectlatest/csp/docbook/DocBook.UI.Page.cls?KEY=EHTTP_inbound#EHTTP_B158193)


**中文编码转码**

如果请求的流里面有中文字符，需要在代码里执行转码，比如下面这个例子，把pInput的流，转码成UTF8,放到Ens.StringRequest里传输。

```java
    set pRequest=##class(Ens.StringRequest).%New()
    set pRequest.StringValue=$ZCVT(pInput.Read(),"I","UTF8")
    set sc= ..SendRequestAsync("Dummy1",pRequest,.pResponse)
```
**有关组件名称和访问的URL**

请求的URL有两个通常的选择：
1. Production里面的业务组件名字和类名称一样

比如上面代码的类是"SEDemo.IO.HTTP.ServiceExample1",如果production的业务服务也是同样的名字，那么调用它的URL就是

        http://hostip:port/csp/healthshare/demo/SEDemo.IO.HTTP.ServiceExample1.cls

其中"/csp/healthshare/demo/"是Web Application的名字。

- 如果上一条不成立。比如写了一个类，用于多个业务服务组件，那么需要组件的名字可以用自己的名字，调用的URL要包含?CfgItem=xxx表示寻找不同的业务组件服务。举例说，还是上面的类，用于添加了两个业务服务组件"httpservice2"和"httpservice3",访问它们的URL就是:
```  
    http://hostip:port/csp/healthshare/demo/SEDemo.IO.HTTP.ServiceExample1.cls?CfgItem=httpservice2
    http://hostip:port/csp/healthshare/demo/SEDemo.IO.HTTP.ServiceExample1.cls?CfgItem=httpservice3
```
>还有这么个操作，就是单独创建一个Web Application, 配置DispatchClass，来接入一个Web服务。我觉得完全没有必要，而且新版本中Web Application的菜单里只剩下了为REST分配分派类的选择，因此这里就不说这个了。





## 第二种： 使用EnsLib.HTTP.GenericService预置业务服务组件

使用预置的，开发好的组件意味着不用写代码，配置一下就可以使用。相应的，灵活性上就差了，大概只适合简单的透传，转发，路由。如果要修改数据包内容，http头内容，编码转换等等，需要要在其他的组件上实现，比如production中消息经过的BP, BO。

EnsLib.HTTP.GenericService的父类是EnsLib.HTTP.Service，因此它也是可以使用Adapter机制或者CSP标准机制。如前面所述，我们需要用CSP机制，调用的URL必须包含**?CfgItem=组件配置名**，比如我在Production里面配置了组件”GenericService1", 那么我访问的请求就应该是：

        http://localhost/csp/healthshare/demo/EnsLib.HTTP.GenericService.cls?CfgItem=GenericHTTPService1

还有，因为我们不修改代码，所以**组件配置项中会有Adapter的配置，比如端口号，SSL配置，直接忽略它们，不用理睬。** 。如果您看到配置项上有”字符集",显示的是UTF-8，可是没起作用，请不要奇怪，这个是Adapter的配置，您用CSP机制时它是不起作用的。 

EnsLib.HTTP.GenericService向其他业务组件发出的Ensemble消息的类型是**EnsLib.HTTP.GenericMessage**。以下是一个POST请求被业务服务组件发送给BO的消息样例。  

```
        <!-- type: EnsLib.HTTP.GenericMessage  id: 531 -->
        <HTTPMessage xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:s="http://www.w3.org/2001/XMLSchema">
        <Stream>{
        âserial_idâ: âC20114062017025â,
        âtake_timeâ: â2019-01-02 15:04:05â,
        âpos_timeâ: â2019-01-02 15:04:05â,
        âtypeâ: â1â,
        âwarnâ: â0â,
        âfile_idâ: â9cd586eef356c71f64b82a190b469e69â,
        âfile_nameâ: âA1012014400596520160714190338.hlyâ,
        âfile_pathâ: â/service/data/TE9100Yâ,
        âbegin_timeâ: â2016-07-04 20:17:21â,
        âend_timeâ: â2016-07-04 20:18:21â,
        âlengthâ: â60â,
        âsizeâ: â9650â,
        âresultâ: âçª¦æ§å¿ç, æ¬æ¬¡å¿çµçæµæªè§å¼å¸¸â
        }</Stream>
        <Type>GB</Type>
        <HTTPHeaders>
        <HTTPHeadersItem HTTPHeadersKey="CSPApplication">/csp/healthshare/demo/</HTTPHeadersItem>
        <HTTPHeadersItem HTTPHeadersKey="CharEncoding" xsi:nil="true"></HTTPHeadersItem>
        <HTTPHeadersItem HTTPHeadersKey="EnsConfigName">GenericHTTPService1</HTTPHeadersItem>
        <HTTPHeadersItem HTTPHeadersKey="HTTPVersion">1.1</HTTPHeadersItem>
        <HTTPHeadersItem HTTPHeadersKey="HttpRequest">POST</HTTPHeadersItem>
        <HTTPHeadersItem HTTPHeadersKey="IParams">1</HTTPHeadersItem>
        <HTTPHeadersItem HTTPHeadersKey="IParams_1">CfgItem=GenericHTTPService1</HTTPHeadersItem>
        <HTTPHeadersItem HTTPHeadersKey="RawParams">CfgItem=GenericHTTPService1</HTTPHeadersItem>
        <HTTPHeadersItem HTTPHeadersKey="TranslationTable">RAW</HTTPHeadersItem>
        <HTTPHeadersItem HTTPHeadersKey="URL">/csp/healthshare/demo/EnsLib.HTTP.GenericService.cls</HTTPHeadersItem>
        <HTTPHeadersItem HTTPHeadersKey="accept">/</HTTPHeadersItem>
        <HTTPHeadersItem HTTPHeadersKey="accept-encoding">gzip, deflate, br</HTTPHeadersItem>
        <HTTPHeadersItem HTTPHeadersKey="authorization">Basic X3N5c3RlbTpTWVM=</HTTPHeadersItem>
        <HTTPHeadersItem HTTPHeadersKey="cache-control">no-cache</HTTPHeadersItem>
        <HTTPHeadersItem HTTPHeadersKey="connection">keep-alive</HTTPHeadersItem>
        <HTTPHeadersItem HTTPHeadersKey="content-length">532</HTTPHeadersItem>
        <HTTPHeadersItem HTTPHeadersKey="content-type">text/plain</HTTPHeadersItem>
        <HTTPHeadersItem HTTPHeadersKey="cookie">CSPSESSIONID-SP-52773-UP-csp-healthshare-=0000000100006fSAbXykPFTd0NFNEoaUH94y07Phmq0V92aeDg; CSPBrowserId=F4nsRY6yDGVipvQ84sDN3w; CSPWSERVERID=I33xYkI3</HTTPHeadersItem>
        <HTTPHeadersItem HTTPHeadersKey="host">172.16.58.200:52773</HTTPHeadersItem>
        <HTTPHeadersItem HTTPHeadersKey="postman-token">0e0bf4a7-868b-4e69-bbac-bfd89909fe99</HTTPHeadersItem>
        <HTTPHeadersItem HTTPHeadersKey="user-agent">PostmanRuntime/7.28.0</HTTPHeadersItem>
        </HTTPHeaders>
        </HTTPMessage>
```



使用这个组件要注意的几点： 
-  上面的数据包里有中文乱码，我是故意这么做的，就是提醒您：强调一下选用这个组件时，如果要修改数据，可以在其他的组件处理。如果我做一个简单的HTTP转发，我可以选用EnsLib.HTTP.GenericService和EnsLib.HTTP.GenericOperation这一对预置组件，这样BS,BO就免开发了。而如果中间要做中文编码的转换，我会插入一个BP, 专门处理EnsLib.HTTP.GenericMessage里的中文转码。

- 组件的"启动标准请求"配置项：要使用CSP Gateway请求机制时应该勾选。

- "持久消息已发送INPROC"配置项（ PersistInProcData）
这个选项是专用于以InProc模式同步调用时是否要持久化Ensemble消息的选项，默认是保存。如果设置成off,那么HealthConnect即不保留消息头，也不保留消息体，在消息查看器里无法查看，也不能重传。

- “保持标准请求分区”配置项
也是用于InProc模式调用，是否保留到BO定义的外部系统的连接。和上面的"持久消息已发送INPROC"配置项一样，很少被用到，保留默认的选中就可以。 

- "没有字符集转换”配置项
控制CSP Gateway是否对%reponse消息里的文本按content-type的类型转码，默认是不勾选，也就是保留CSP Gateway的转码功能。

- ”One-Way"配置项
如果客户端不期待响应消息，那么选中这个选项后，EnsLib.HTTP.GenericService收到请求转发的同时会送一个Http状态码202，意思是”服务器已接受请求，但尚未处理“. 默认是不选中这个配置。

有什么问题和指教，请留言或者发我邮件。

----
END  
-----









 分清什么任务适合HealthConnect上实现或者是在Web服务器(IIS,Apache,Nginx...)上实现。如果一个任务Web服务器可以实现，通常要比在HealthConnect上用ObjectScirpt写代码要容易的多。


# 开发前的准备，HTTP知识

绝对的URL，或者叫Fully Qualified Absolute URL的结构如下面的样子：其中Scheme是协议名称，比如http,ftp,javascript等等。在HealthConnect的场景里，基本上只是要处理http和https.后面login.password@叫做credentials to access the resource,http协议时基本不会用到。再后面是服务器地址和端口号address和port;资源路径”/path/to/resource"; 请求参数“?query_string"；而"#fragment"是又一个不太会用到的选项。  

    Scheme://login.password@address:port/path/to/resource?query_string#fragment  

对于HealthConnect的开发者，常规见到的URL类似这样： 

    http://ipaddr:port/WebApplicationName/ClassName.cls?Item=abc

再强调一遍: 上面的port是发给Web服务器的端口号，生产环境不建议使用HealthConnect的PWS端口57773，或者自定义的TCP监听端口。安全和可靠的HTTP服务应该是从Web服务器接收HTTP请求，经过InterSystems的WebGateway发给HealthConnect配置的Web Application。有关Web Appliaction的配置见(...)

## HTTP消息头

下面是一个POST的HTTP请求响应的例子，如果对HTTP头的字段的含义不熟悉，可以参考这个网址: [HTTP头](https://www.jianshu.com/p/6f29fcf1a6b3)

    //请求头  
    POST /fuzzy_bunnies/bunny_dispenser.php  
    HTTP/1.1 Host: www.fuzzybunnies.com  
    User-Agent: Bunny-Browser/1.7  
    Content-Type: text/plain   
    Content-Length: 17  
    Referer: http://www.fuzzybunnies.com/main.html    
    //响应头
    HTTP/1.1 200 OK 
    Server: Bunny-Server/0.9.2 
    Content-Type: text/plain 
    Connection: close



//nuodar code

   Class YSKJ.Reform.BS.AppointmentRegisterBS Extends EnsLib.HTTP.Service
    {

    Parameter ADAPTER;
    
    /// 虚拟路径第一层IP:port/directorys
    /// Parameter EnsServicePrefix = "|yskj";
    Method OnProcessInput(pInput As %Stream.Object, Output pOutput As %Stream.Object) As %Status
    {
    
        /// do pResponse.Result.Write($ZCVT(tResponse.Data.Read(),"I","UTF8",handler))
    set ipput = pInput.Read(,.tSC)
    $$$LOGINFO("入参："_$ZCVT(ipput,"I","UTF8",handler))
    $$$LOGINFO("状态："_tSC)
    if $$$ISERR(tSC)
    do $System.Status.DisplayError(tSC)
    ///[{"key":"Timestamp","value":"1604563095542","description":"","type":"text","enabled":true}]
    set username = pInput.Attributes("user-name")
    set timestamp = pInput.Attributes("timestamp") 
    set sign = pInput.Attributes("sign")  
    Set tRequest = ##class(YSKJ.Reform.MSG.AppointmentRegister.Req.AppointmentRegisterReq).%New()
    Set tRequest.Input = $ZCVT(ipput,"I","UTF8",handler)
    Set tRequest.username = username
    Set tRequest.timestamp = timestamp
    Set tRequest.sign = sign
    Set tSC = ..SendRequestSync("YSKJ.Reform.BO.AppointmentRegisterBO",tRequest,.tRespone)
    $$$LOGINFO("返回："_tRespone.Result)
    Set tRespone11 = ##class(YSKJ.Reform.MSG.AppointmentRegister.Req.AppointmentRegisterResp).%New()
    set tRespone11.Result = tRespone.Result
    $$$LOGINFO("返参："_tRespone11.Result.Read(,.tsc))
    Set pOutput = tRespone11.Result
    Quit tSC
    }
    
    }



