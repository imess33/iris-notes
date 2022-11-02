# 从unixodbc连接IRIS数据库

从2021.2开始， IRIS也支持DB-api了， 因此应该很少有机会以本文的方式连接IRIS了。 至少python开发者不用了。

### 1. 安装unixODBC

http://www.unixodbc.org/ 网站提供下载软件， 还包括各种数据库驱动的下载（但没有Cache'和IRIS的驱动，他们的驱动您得去InterSystems网站下载). 对于不同的linux版本或者Mac, 下载相应的版本并安装。

> Mac预置了iODBC， 但性能比UnixODBC差

在Mac上安装unixodb有更简单的办法。

```bash
brew install unixodbc
odbcinst -j
```

安装结束可以通过以下命令查询unixodbc的安装结果（CentOS的执行结果）

```bash
[root@zabbix irissys]# odbcinst -j
unixODBC 2.3.1
DRIVERS............: /etc/odbcinst.ini
SYSTEM DATA SOURCES: /etc/odbc.ini
FILE DATA SOURCES..: /etc/ODBCDataSources
USER DATA SOURCES..: /root/.odbc.ini
SQLULEN Size.......: 8
SQLLEN Size........: 8
SQLSETPOSIROW Size.: 8
[root@zabbix irissys]#
```

"odbcinst"是unixodbc的管理命令，不仅仅是安装，添加driver, DSN也都是这个命令。不熟悉格式可以直接敲命令查看帮助。 

### 2. 安装IRIS ODBC Driver

https://docs.intersystems.com/irislatest/csp/docbook/Doc.View.cls?KEY=BNETODBC_unixinst

上面说到， 您需要从InterSystems得到IRIS ODBC的驱动软件，比如我这里用这个RedHat的安装包（**ODBC-2021.1.0.205.0-lnxrhx64.tar.gz**）演示下面的步骤。 

```bash
[root@zabbix irissys]# pwd
/usr/irissys
[root@zabbix irissys]# tar -xf ODBC-2021.1.0.205.0-lnxrhx64.tar.gz
[root@zabbix irissys]# ls
bin  dev  ODBC-2021.1.0.205.0-lnxrhx64.tar.gz  ODBCinstall
[root@zabbix irissys]# ./ODBCinstall
    Creating irisodbc.ini ...
 Done setting up ODBC and SQLGateway!
 [root@zabbix irissys]# ls /usr/irissys/bin
irisconnect.so  libirisodbc35.so    libirisodbcur6435.so  libodbc.so.2      odbcgatewayiw.so  odbcgatewayur64.so libiodbc.so     libirisodbciw35.so  libodbc.so libodbc.so.2.0.0  odbcgateway.so
[root@zabbix irissys]#
```

这时候IRIS ODBC的驱动程序已经安装在操作系统上了， 关于各个.so文件的用法和区别，请参考在线文档。简单的说, libirisodbcu*是为unixodbc使用的， libirisodbci开头的是为iODBC使用的。 这里我要使用的是**libirisodbcur6435.so**

注意一点： 选择"/usr/irissys"作为安装目录不是必须的。但因为后面步骤里面将irisodbc驱动注册在unixodbc的脚本里面用的这个目录，所以为了省事，我也选择创建并安装驱动在这里。 

> 在Mac OS，安装到/usr/local/etc/irisodbc

### 3. 在unixodbc注册iris driver, 

```bash
[root@zabbix irissys]# cd dev/odbc/redist/unixodbc/
[root@zabbix unixodbc]# pwd
/usr/irissys/dev/odbc/redist/unixodbc
[root@zabbix unixodbc]# ls
include  iusql  libodbcinst.so.2      libodbc.so    odbcinst  readme.txt isql     libodbcinst.so  libodbcinst.so.2.0.0  odbc.ini_unixODBCtemplate  odbcinst.ini_unixODBCtemplate
[root@zabbix unixodbc]# odbcinst -i -d -f odbcinst.ini_unixODBCtemplate
[root@zabbix unixodbc]# cat /etc/odbcinst.ini
      ...
      [InterSystems ODBC]
      UsageCount=1
      Driver=/usr/irissys/bin/libirisodbcur6435.so
      Setup=/usr/irissys/bin/libirisodbcur6435.so
      SQLLevel=1
      FileUsage=0
      DriverODBCVer=02.10
      ConnectFunctions=YYN
      APILevel=1
      DEBUG=1
      CPTimeout=<not pooled>
[root@zabbix unixodbc]# 
```

当检查“/etc/odbcinst.ini“文件时，应该能发现里面有”InterSystems ODBC"的驱动记录。确认里面配置的.so文件的名字和路径名称。这个记录来自“odbcinst.ini_unixODBCtemplate”文件，里面的内容可能适合以往的版本，所以您可能要手工修改/etc/odbcinst.ini文件。直接改就可以， 不需要做任何重启的动作。 



### 4. 创建DSN到IRIS Server 

```bash
[root@zabbix unixodbc]# odbcinst -i -s -l -f odbc.ini_unixODBCtemplate
[root@zabbix unixodbc]# cat /etc/odbc.ini
		...
    [user]
    Driver=InterSystems ODBC
    Protocol=TCP
    Host=localhost
    Port=51773
    Namespace=USER
    UID=_system
    Password=sys
    Description=User namespace
    Query Timeout=0
    Static Cursors=0
[root@zabbix unixodbc]# 
```

"odbcinst -i -s -l -f odbc.ini_unixODBCtemplate"命令中的“-l"是说创建一个System DSN, 通常这是最合适的旋转， 如果要创建Local  DSN，命令要改成“odbcinst -i -s -h -f odbc.ini_unixODBCtemplate”。

这时您要检查并手工修改上面记录里的配置， 包括IP, port, Namespace, UID, Password, 确认您在连接一个正工作的IRIS Server的数据库。或者， 您可以手工添加更多的DSN。

最后， 尝试连接您的库， 比如下面时个成功连接的样子：

```bash 
[root@zabbix unixodbc]# isql user
+---------------------------------------+
| Connected!                            |
|                                       |
| sql-statement                         |
| help [tablename]                      |
| quit                                  |
|                                       |
+---------------------------------------+
SQL>
```



### End

