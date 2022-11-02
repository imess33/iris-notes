# Web Gateway 和 Web Application

尽管Cache/IRIS可以监听TCP接口，处理HTTP请求消息，但从性能和安全的考虑，推荐的生产环境的部署模式是“CSP 机制”。 

CSP是Caché Server Page的缩写，是InterSystems的前端页面技术。

CSP机制是指： HTTP请求经过外部连接的Web Server, 比如Apache Server, IIS, Nginx，从TCP 2188接口发送给Caché /IRIS. 为了让这些Web服务器将请求转发给Caché/IRIS, 需要在服务器上安装InterSystems的模块，它们被称为InterSystems Web Gateway(早期版本称为CSP Gateway).

在Cache'/IRIS内部，收到的请求由Web Application处理。

图1 

因此，在Ensemble的Production中，在创建各种Web服务(SOAP, REST, HTTP)时， 先要保证Web Server能正确的转发请求给Web Application, 而Web Application配置正确。 

***PWS(Private Web Server) ***

IRIS的安装过程会安装一个私有的Apache Web服务器，被称作PWS。它的作用有两个：支持访问维护页面;给一个测试环境提供测试Web服务的能力。它的默认配置端口是57772。

**它不能在生产环境中承载Web服务**

更多的详细内容，包括怎么安装配置Web Gateway, 请参见： [Web Gateway介绍](https://cn.community.intersystems.com/post/webgateway%E7%B3%BB%E5%88%971-web-gateway%E4%BB%8B%E7%BB%8D)



## 配置Web Application

*TBD*



