# 开发REST服务- Spec First



Spec First, 也就是从接口规范出发的开发方式有诸多好处，一个是简便快捷，可以从接口定义生成接口代码；二是方便管理，无论是接口定义的准确还是接口文档的自动生成，比从代码开始开发都有优势。其他的优点还有开发流程上的灵活等等。我们先从OAS，也就是接口规范开始了解。 



## OAS(OpenAPI Specification)和Swagger

(https://swagger.io/solutions/getting-started-with-oas/)

下面是一个最简单的OAS定义的接口。

```json
{
  "swagger": "2.0",
  "basePath": "/helloworld",
  "info": {"title": "Hello World service","version": "1.0.0"},
  "paths": 
  {
    "/": {
      "get":{ "description": "returns 'hello,world'",
              "operationId": "Test",
              "responses": { "200": {"description": "success",}},
              "summary": "Return hello world message.",
              "produces": [ "text/plain"]
            }
          }
  }
}
```

编写接口规范的最常用的工具是Swagger Editor，[官方网址](https://swagger.io/tools/swagger-editor/download/)提供各个操作系统上的应用下载，和Cloud服务入口。如果喜欢使用docker container, 可以到[这里下载](https://hub.docker.com/r/swaggerapi/swagger-editor/)。

> Run Container with "docker run -d -p 80:8080 swaggerapi/swagger-editor " and visit http://localhost



## 开发的步骤

### Step1: 使用Swagger Editor编写接口规范

除了上面的示例， 可以参考hccapi.yaml, 或者api.mgmnt中的json格式

**要导入IRIS，OAS需要保存为json格式。**

## Step 2：导入IRIS，检查创建的类

```bash
DEMO>do ^%REST
 
REST Command Line Interface (CLI) helps you CREATE or DELETE a REST application.
Enter an application name or (L)ist all REST applications (L): HCC.api
REST application not found: HCC.api
Do you want to create a new REST application? Y or N (Y): y
 
File path or absolute URL of a swagger document.
If no document specified, then create an empty application.
OpenAPI 2.0 swagger: c:\temp\hccapi.json
 
OpenAPI 2.0 swagger document: c:\temp\hccapi.json
Confirm operation, Y or N (Y): y
 
-----Creating REST application: HCC.api-----
CREATE HCC.api.spec
GENERATE HCC.api.disp
CREATE HCC.api.impl
REST application successfully created.
 
Create a web application for the REST application? Y or N (Y): y
Specify a web application name beginning with a single '/'. Default is /csp/HCC/api
Web application name: /api/hcc
 
-----Deploying REST application: HCC.api-----
Application HCC.api deployed to /api/hcc
 
DEMO>
```

接下去， 配置web application:

- 注意权限， 尤其是unknown user, 需要有权限访问数据库

注意： 

生成3个文件， 其中.disp文件因为是从.json生成的， vscode里面找不到。 :(















## 练习

创建一个接口接收POST, 内容为：

> [hl7- fhir: patient resource](https://www.hl7.org/fhir/patient.html)

```json
{
    "resourceType": "Patient",
    "id": "5",
    "identifier": [
        {
            "system": "2.16.840.1.113883.2.23.1.9.1",
            "value": "110101200301120019"
        },
        {
            "system": "2.16.840.1.113883.2.23.1.9.2",
            "value": "100000000000"
        }
    ],
    "name": [
        {
            "text": "刘康",
            "family": "刘",
            "given": [
                "康"
            ]
        }
    ],
    "gender": "male",
    "birthDate": "2003-01-12",
    "address": [
        {
            "use": "home",
            "text": "北京市东城区景山前街4号",
            "line": [
                "景山前街4号"
            ],
            "city": "北京市",
            "district": "东城区",
            "state": "北京",
            "postalCode": "100010"
        }
    ],
}
```





##class(%REST.API).CreateApplication()



生成3个class



>REF: 
>
>HealthConnect提供的REST服务
>
>/api/mgmnt/v2
>
>/api/mgmnt/v1/user/restapps
>
>
>
>

