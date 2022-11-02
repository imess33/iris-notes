# 2021-09-21



新星提供的一个wsdl, 无法使用soap add-in生成web service, 但在客户的环境里面可以， 虽然生成的服务会报告‘字符串超长’的错误。

客户用url还是file都可以导入，区别是他们和原始服务有连接。



下载的wsdl有这样一句：

```xml
<wsdl:import location="http://192.168.200.159:8080/platform/ws/patient?wsdl=platform.wsdl" namespace="http://webservice">
    </wsdl:import>
```

在我的机器上会造成连接不上端口的错误。

如果把它去掉， 或者改成自己的地址，会报错：

```
Generator Output in Namespace demo:
Creating classes...
WSDL: C:\temp\test.wsdl

ERROR #6411: Element ''definitions' or 'schema'' is missing
Completed at 2021-09-22 17:33:41
```

查资料如下： 

import元素使得可以在当前的WSDL文档中使用其他WSDL文档中指定的命名空间中的定义元素。本例子中没有使用import元素。通常在用户希望模块化WSDL文档的时候，该功能是非常有效果的。 
import的格式如下：

<wsdl:import namespace="http://xxx.xxx.xxx/xxx/xxx" location="http://xxx.xxx.xxx/xxx/xxx.wsdl"/>

必须有namespace属性和location属性： 
1.namespace属性：值必须与正导入的WSDL文档中声明的targetNamespace相匹配； 
2.location属性：必须指向一个实际的WSDL文档，并且该文档不能为空。

所以导入不成功是正常的。 



## 第2个发现



用wsdl生成web service, 得到的是%SOAP.WebService, 而不是ensemble组件， 至于怎么做成ensemble组件， 还得和新星讨论。 



