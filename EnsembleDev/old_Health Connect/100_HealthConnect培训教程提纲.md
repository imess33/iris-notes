# HealthConnect培训教程提纲



# Lesson 1: HealthConnect介绍

- HealthConnect的产品分类，市场定位，特点
- 本身的架构，和IRIS的关系，InterSystems其他的产品
- 有关安装， Supported platform
- HA, license, Web Gateway
- 学习曲线，学习路径，
- 项目开发的流程，
- InterSystems提供的支持

## Lesson 2: Production

- 创建Namespace , database

- 创建Production, 介绍各种BusinessHost, 添加组件，例如HL7的file Output, 组件消息的发送， Message Trace, Event Log查看，组件设置项； 设置TCP接收，让学生发送TCP数据给教员；查看Queue; 加入?BS, 发送消息给教员。

  

## Lesson 3:  Ensemble Message

- 查看以前生成的消息表格，了解EnsembleHeader, EnsembleBody
- 消息的管理，查看，删除
- ObjectScript: Class definition, %Persistent, %SerialObject, %Registered, 消息到表的映射
- method, relationship??
- Ens.Request, Ens.Response, Ens.StreamContainer, Ens.StringContainer
- 自定义的消息
- 字符串的处理
- 流的处理

## Lesson 4: Business Host和适配器

- BO - xdata, using Outbound Adapter
- BS - using InboundAdapter, OnInit(), OnProcessInput()
- ConfigItem（组件配置项)
- TCP outbound Adapter
- TCP inbound Adapter
- Reply Code Action

## Lesson 8: Message Router



## Lesson: Data Transmission

- lookup table
- FunctionSet

## Lesson 5: SQL Adapter (Optional)

- Outbound Adapter
- Inbound Adapter



## Lesson 5 COS 

- ObjectScript （60m)
  - 练习: command: write, set, quit/return, do, 时间，
  - 字符串，流
- Class: %Registered, %Persistent, %SerialObject (30m)
  - 演示：创建各类class, 展现SQL, 简单介绍Global 

## Lesson 6: Web Client

使用Studio Wizard创建Web Client (30m)

## Lesson 7: Web Gateway and Web service

- Web Gateway的机制，配置 (20m)
- Web Application的配置 (15m)
  - 练习： 创建单独的Web Application, 
- 实现简单的SOAP Service, 解释最重要的几个要点（40m)
  - 练习: 创建互联互通服务端，入参为action:stirng, message:%Stream, 查看WSDL
  - 定义服务类。添加到Production

## Lesson 8: XML消息的处理

- object-xml correlation (60m)
- 使用XPath（30m)
  - 练习： 生成一个patient类， 从中取值打印到屏幕上。
  - 配置BPL的xpath
- 使用VDoc (Embedded Schema) (120m)
- xml node validataion, modify and xslt (skip)

## Lesson 9: HTTP/REST Client

- REST and HTTP Client (100m)

## Lesson 10: HTTP/REST Service in Ensemble

- HTTP Service(30m)
  - 练习 ：开发最简单的HTTP服务端， 接受body, 返回200OK和简单的字符串，使用EnsLib.HTTP.Service

- REST Service手工开发( 100m)
  - 练习 ： 开发一个简单的Rest服务，
- REST Service Spec-First (100m)
- JSON消息的处理

## Lesson 11: Passthrough and ESB

- passthrough and ESB (120m)

## BPL

- call, assign, if/switch, for, trace
- pubsub
- 在BPL/DTL中使用变量(Optional)
- 复杂BPL设计

## 流程控制

- Reply Code Action
- Pool Size 
- Queue management 

## 错误处理

- COS错误处理
- BPL error handling
- alert manager
- bad message handling

## 操作维护

- System and Production Dashboard
- Activity Data
- alert and notification
- 消息删除
- 导入导出
- 部署模式

## 其他



## 开发的过程以及Best Practice

- 怎么分配instance, namespace

- 要不要保留消息，为什么要经过HC
  - pass-through & esb
  - 统计业务流量
  - 对外暴露

- 保留消息的话
  - 为什么目的？分析还是troubleshooting
  - 保留多久
  - 怎么构造消息？ 尤其是各种类的组合，要不要用serial object, 要不要虚拟文档





