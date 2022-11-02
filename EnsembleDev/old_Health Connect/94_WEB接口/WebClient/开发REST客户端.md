# 开发Ensemble REST客户端



让我们以一个简单的公共REST服务的资源来开始Ensemble的REST客户端开发。

[Postman Echo Service](https://docs.postman-echo.com/?version=latest)是postman提供的用来测试的一个echo服务，它把你发的请求‘反射’回来。先来看看最简单HTTP的请求：

```
CNMBPHMA:~ hma$ {"args":{"foo1":"bar1"},"headers":{"x-forwarded-proto":"http","x-forwarded-port":"80","host":"postman-echo.com","x-amzn-trace-id":"Root=1-60eeac1a-566d2c5a309a92fe77e411c3","user-agent":"curl/7.54.0","accept":"*/*"},"url":"http://postman-echo.com/get?foo1=bar1"}
[1]+  Done                    curl http://postman-echo.com/get?foo1=bar1
CNMBPHMA:~ hma$
```



Ensemble中的REST客户端的开发使用[EnsLib.REST.Operation](https://docs.intersystems.com/healthconnectlatest/csp/documatic/%25CSP.Documatic.cls?&LIBRARY=ENSLIB&CLASSNAME=EnsLib.REST.Operation)。它使用了HTTP出向适配器，并继承了Ens.Util.JSON。HTTP Adapter最常用的method包扩GetURL(), PostURL()...

- 代码中Echo()通过检查发来的字符串，确认是GET方法还是POST方法，然后调用GetURL()或者PostURL()
- 从echo service返回的结果在tHttpResponse.Data里面， 是一个流。

```
Class SEDemo.IO.REST.EchoOperation Extends EnsLib.REST.Operation
{
Parameter INVOCATION = "Queue";

/// https://postman-echo.com/get?foo1=bar1&foo2=bar2
/// GET: "get?id=123" or "post?id=123"
/// POST Raw Text: POST /hi/there?hand=wave
/// POST Form Data: 
Method Echo(pRequest As Ens.StringContainer, Output pResponse As Ens.StringContainer) As %Status
{
     Set tURL=..Adapter.URL_pRequest.StringValue
     if (pRequest.StringValue["get") { 	
      		Set tSC= ..Adapter.GetURL(tURL,.tHttpResponse)
      	}elseif (pRequest.StringValue["post"){
	      set tFormVar="foo, bar"
	      set tfoo=1, tbar=2
	      Set tSC= ..Adapter.PostURL(tURL,.tHttpResponse,tFormVar,tfoo,tbar)
     }
     //构建Ensemble响应消息
     If $IsObject(tHttpResponse.Data) 
     {  
    		set pResponse=##class(Ens.StringContainer).%New()
				set pResponse.StringValue=tHttpResponse.Data.Read()	
      }
   	Quit tSC
}

XData MessageMap
{	<MapItems>
    <MapItem MessageType="Ens.StringContainer"> 
    	<Method>Echo</Method>
  	</MapItem>
  </MapItems>
}
}

```

