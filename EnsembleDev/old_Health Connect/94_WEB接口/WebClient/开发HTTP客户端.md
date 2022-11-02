# 开发Ensemble HTTP客户端



> REF : https://docs.intersystems.com/healthconnectlatest/csp/docbook/Doc.View.cls?KEY=EHTTP_outbound

# 使用[EnsLib.HTTP.OutboundAdapter](https://docs.intersystems.com/healthconnectlatest/csp/documatic/%25CSP.Documatic.cls?LIBRARY=ENSLIB&CLASSNAME=EnsLib.HTTP.OutboundAdapter)创建BO

请看下面的示意代码

```
Class SEDemo.IO.HTTP.BOExample Extends Ens.BusinessOperation
{

Parameter ADAPTER = "EnsLib.HTTP.OutboundAdapter";

Property Adapter As EnsLib.HTTP.OutboundAdapter;

Parameter INVOCATION = "Queue";

Method HTTPGetSample(Output pResponse As Ens.StringResponse) As %Status
{
 	set status=..Adapter.Get(.phttpResponse)
	do phttpResponse.OutputToDevice()
	//创建返回的Ensemble消息
	...
	Quit $$$OK
}

Method HTTPPostSample(pRequest As Ens.StringRequest, Output pResponse As Ens.StringContainer) As %Status
{
	set Url="http://postman-echo.com/post"

	///send form data
	set tFormVar="patientid,姓名"
 	set status=..Adapter.PostURL(Url,.tResponse,tFormVar,"123","李岩")
	
	set pResponse=##class(Ens.StringContainer).%New()
	
	///如果出错显示内部错误信息，否则显示reponse消息
	quit:'status status
	//构建Ensemble Response消息
	set pResponse=##class(Ens.StringContainer).%New()
	set pResponse.StringValue=tResponse.Data.Read()	
	
	Quit $$$OK
}

}
```

## HTTP请求的发送

提供以下方法发送HTTP请求: Get(), GetFormData(), PostFormData(), PostFormDataArray()等等 。

HTTP请求使用IRIS的default character coding, 如果要修改encoding, 需要修改HTTP request. 

### 响应消息%Net.HttpResponse的处理

Property 

- Data as %RawString
  The stream or a string contains all the data sent by the web server after the HTTP headers. You can test if this is a stream with $isobject(response.Data) and if it is not a stream then it is a string with the data in it.
- Header as %String
- StatusCode as %Integer
- StatusLine as %String
- ContentType as %String
  Method()
- OutputToDevice()

最简单的读消息体

set pResponse.StringValue=tResponse.Data.Read()

## 适配器属性的配置

你可以在组件配置项中配置， [在线文档:Settings for the HTTP Outbound Adapter](https://docs.intersystems.com/healthconnectlatest/csp/docbook/Doc.View.cls?KEY=EHTTP_settings_outbound)

- 目标IP, 端口号(HTTPServer, HTTPPort)
- URL
- TLS
- Credentials
- ConnectTimeout (default 5s)
- debug(0,1,2)

ConnectionTimeOut

默认值是5秒，如果这段时间连接没有打开，Adapter会重试，直到FailureTimeout除以Retry Interval.
同样情况还有ResponseTimeOut, 默认值是30秒

知道了怎么发送HTTP请求的基本方法， 我们来看看下面几个问题

### 发送表单数据

​	set tFormVar="patientid,姓名"
 	set status=..Adapter.PostURL(Url,.tResponse,tFormVar,"123","李岩")

或者

```
set tFormVar="USER,ROLE,PERIOD"
set tUserID=pRequest.UserID
set tRole=pRequest.UserRole
set tED=pRequest.EffectiveDate
set tSC=..Adapter.Get(.tResponse,tFormVar,tUserID,tRole,tED)
```

### 发送请求的消息体

```
set tsc = ..Adapter.Post(.tResponse,,pRequest.MessageStream)
```

### 发送自定义的消息头

```
Method Post(Output pHttpResponse As %Net.HttpResponse,pFormVarNames As %String = "",pData...) As %Status
```



## 练习：

[Postman Echo Service](https://docs.postman-echo.com/?version=latest)是postman提供的用来测试的一个echo服务，它把你发的请求‘反射’回来。先来看看最简单HTTP的请求：

- GET

```
CNMBPHMA:~ hma$ curl http://postman-echo.com/get
{"args":{},"headers":{"x-forwarded-proto":"http","x-forwarded-port":"80","host":"postman-echo.com","x-amzn-trace-id":"Root=1-60f2ed16-2eaeb65a6df1f4985334df8e","user-agent":"curl/7.54.0","accept":"*/*"},"url":"http://postman-echo.com/get"}CNMBPHMA:~ hma$

CNMBPHMA:~ hma$ curl --location --request GET 'https://postman-echo.com/get?foo1=bar1&foo2=bar2'
{"args":{"foo1":"bar1","foo2":"bar2"},"headers":{"x-forwarded-proto":"https","x-forwarded-port":"443","host":"postman-echo.com","x-amzn-trace-id":"Root=1-60f2edb9-18b665443a9d2902302b5e22","user-agent":"curl/7.54.0","accept":"*/*"},"url":"https://postman-echo.com/get?foo1=bar1&foo2=bar2"}CNMBPHMA:~ hma$
```

- POST （from postman)

  - URL: http://postman-echo.com/post
  - raw: /hi/there?hand=wave

  ```
  {
      "args": {},
      "data": "/hi/there?hand=wave",
      "files": {},
      "form": {},
      "headers": {
          "x-forwarded-proto": "http",
          "x-forwarded-port": "80",
          "host": "postman-echo.com",
          "x-amzn-trace-id": "Root=1-60f2eee4-201e679d43d9f6ae1bc9ec98",
          "content-length": "19",
          "authorization": "Basic c3VwZXJ1c2VyOlNZUw==",
          "content-type": "text/plain",
          "user-agent": "PostmanRuntime/7.26.8",
          "accept": "*/*",
          "cache-control": "no-cache",
          "postman-token": "0cf895f5-b024-4020-92bf-6303a497eef7",
          "accept-encoding": "gzip, deflate, br",
          "cookie": "sails.sid=s%3A31gSaWKIQFEm0slyrKyEt5MkmqTjrCyc.9wH8J7z%2BqxfYp6go4hZF4qd6zEURoXvL18lZc6jiaAg"
      },
      "json": null,
      "url": "http://postman-echo.com/post"
  }
  ```

  - formed-data

  ```
  {
      "args": {},
      "data": {},
      "files": {},
      "form": {
          "name ": "lisi ",
          "id ": "10001"
      },
      "headers": {
         ...
      "json": null,
      "url": "http://postman-echo.com/post"
  }
  ```

# the END







诺大：

    Class YSKJ.Reform.BO.AppointmentRegisterBO Extends Ens.BusinessOperation
    {
    
    Parameter ADAPTER = "EnsLib.HTTP.OutboundAdapter";
    
    Property Adapter As EnsLib.HTTP.OutboundAdapter;
    
    Parameter INVOCATION = "Queue";
    
    Method AppointmentRegister(pRequest As YSKJ.Reform.MSG.AppointmentRegister.Req.AppointmentRegisterReq, Output pResponse As YSKJ.Reform.MSG.AppointmentRegister.Req.AppointmentRegisterResp) As %Status
    {
    try{
    set STARTTIME = $ZDATETIME($NOW(),3,1,3)
      set pResponse=##class(YSKJ.Reform.MSG.AppointmentRegister.Req.AppointmentRegisterResp).%New()
    set tResponse= ##class(%Net.HttpResponse).%New()
    set tRequest= ##class(%Net.HttpRequest).%New()
    set url = ..Adapter.URL
      set tsc = tRequest.SetHeader("User-Name",pRequest.username)
    set tsc = tRequest.SetHeader("Timestamp",pRequest.timestamp)
    set tsc = tRequest.SetHeader("Sign",pRequest.sign)
    Do tRequest.EntityBody.Write(pRequest.Input)
    //set tsc= ..Adapter.Post(.tResponse,,pRequest.Input)
    set tsc = ..Adapter.SendFormDataArray(.tResponse,"POST",tRequest,,,url)
    
    set len = tResponse.Data.SizeGet()
        While (tResponse.Data.AtEnd = 0) {
          do pResponse.Result.Write($ZCVT(tResponse.Data.Read(),"I","UTF8",handler))
        }
        
            set ENDTIME = $ZDATETIME($NOW(),3,1,3)
      Set status=pRequest.XMLExportToString(.resStr) 
      Set status2=pResponse.XMLExportToString(.respStr)
    s syu = ##class(YSKJ.LOGWS.MsgLogUtil).AddLog(resStr,respStr,"YSKJ.Reform.BS.AppointmentRegisterBS_MedicalCardRegistration","YSKJ.Reform.BO.AppointmentRegisterBO_AppointmentRegister","A","0",STARTTIME,ENDTIME,"WS","READ",.pRequest2) 
    //$$$LOGINFO("5555")
    s tSC11=..SendRequestAsync("YSKJ.LOGWS.AddMsgLog",pRequest2)
        }catch err{
        $$$LOGERROR(err.DisplayString())
        set ENDTIME = $ZDATETIME($NOW(),3,1,3)
      Set status=pRequest.XMLExportToString(.resStr)
    
    s syu = ##class(YSKJ.LOGWS.MsgLogUtil).AddLog(resStr,err.DisplayString(),"YSKJ.Reform.BS.AppointmentRegisterBS_MedicalCardRegistration","YSKJ.Reform.BO.AppointmentRegisterBO_AppointmentRegister","A","1",STARTTIME,ENDTIME,"WS","READ",