# Linux系统IRIS安装总结



本文档只是给您一个大致的概念，方便您了解IRIS安装的方方面面。并非官方安装手册或者安装建议。生产环境的安装请仔细研究并参照[IRIS官方的安装文档](https://docs.intersystems.com/irislatest/csp/docbook/DocBook.UI.Page.cls?KEY=AB_install#AB_install_Linux)



### 操作系统的选择

IRIS支持多种Linux系统。详细的列表请参见各个版本的支持列表，比如这个是最新的[2021IRIS的支持列表](https://docs.intersystems.com/irislatest/csp/docbook/DocBook.UI.Page.cls?KEY=ISP_technologies#ISP_platforms_server)。需要注意的是, CentOS只支持作为测试系统, 并不支持生产系统的安装。 

官方文档是以Redhat为主介绍的IRIS安装， 这是IRIS在生产环境中最常用的Linux系统之一。另一个Ubantu， InterSystems提供的docker版本是安装在Ubantu上的。

我自己测试的时候喜欢使用CentOS, 所以后面的安装步骤，使用的CentOS上的记录。



### 服务器的配置

对于生产系统，安装前需要一个硬件配置方案，这个方案通常是项目的实施部门根据最终用户的需求设计的。

对于测试系统，只有一个最小的磁盘的要求：1.5G。无论是实体服务器， 虚拟服务器，或者docker container，根据您要测试的内容， 一切以方便为准。如果不做性能测试，我个人经常使用的CentOS虚拟机是2核CPU, 2G或4G内存，没有运行不流畅的问题。 

### 磁盘分区

标准的Linux磁盘分区并不适合IRIS的生产运行。哪怕要创建一个尽量接近生产的测试环境，建议您安装软件前要有一个完整的磁盘分区规划。在官方文档中，创建IRIS服务器的磁盘分区有以下原则：

1. **IRIS程序，客户的数据库，首选Journal，备选Journal应该是分开的标准分区或者逻辑盘。所以至少要4个分区/逻辑盘**。

   您也可能会喜欢再给backup文件和其他的比如IRIS使用的web文件添加更多的逻辑盘。

2. 文件系统方面，IRIS程序，客户的数据库推荐使用xfs文件格式，**journal盘推荐ext4格式**。( 经过专家确认，这个推荐不属于强烈推荐，更不是强制要求。大多数情况下使用xfs格式没问题。只是在大项目中，比如当您的Journal盘需要分配1TB的空间，特别是使用ECP服务器的情况下， ext4会因为更低的延时带来相对好的性能表现）

以下是一个测试系统中的分区配置，仅仅是示意。

```sh
[root@MyCentOS7 ~]# cat /etc/fstab
/dev/mapper/centos_hmscentos7-root /               	xfs     defaults        	0 0
UUID=1d1b46a2-d66d-4346-8709-e70854088b11 /boot    	xfs     defaults        	0 0
/dev/mapper/vg_iris-isc_data /isc/data             	xfs     defaults        	0 0
/dev/mapper/vg_iris-isc_iris /isc/iris             	xfs     defaults        	0 0
UUID=c9ce6576-b469-4c74-8867-fe5b5060ad62 /isc/j1 		ext4    defaults				1 2
UUID=0be8a20a-b76c-4a17-9018-ecea1964b73b /isc/j2 		ext4    defaults				1 2
/dev/mapper/centos_hmscentos7-swap swap               swap    defaults        	0 0
[root@hmsCentOS7 ~]#
```



再来看一个生产环境的配置实例, 其中/isc/iris是iris安装路径：下面放系统程序，系统使用的数据库，以及和系统工作相关的文件等等，通常分配100-200G。其他的逻辑盘按客户需要分配。**SWAP, /boot, /home, /var等等的配置按Linux操作系统的推荐及客户的方案配置**。

| **Mount Point** | **VG**  | **LV**    | Size | **File System** | **Mount Options** |
| :-------------- | :------ | :-------- | :--- | --------------- | :---------------- |
| /isc/iris       | vg_iris | lv_iris   | 200G | XFS             | 默认, nobarrier   |
| /isc/db         | vg_iris | lv_irisdb | 1TB  | XFS             | 默认, nobarrier   |
| /isc/jrnpri     |         |           | 500G | EXT4            | 默认              |
| /isc/jrnsec     |         |           | 500G | EXT4            | 默认              |

在上面的方案中， iris系统自带的数据库IRISTEMP是放在/isc/iris下的。 IRISTEMP正常情况下不会很大，但如果应用的设计很特别，或者出了什么问题，分配了200G大小的/isc/iris可能不足够。如果生产环境发现IRISTEMP有过大的风险，您可能需要单独给IRISTEMP分配一个逻辑盘。  



###优化服务器内存配置

以下的对服务器内存的设置可以提高IRIS的性能。 

**使用HugePage内存**

IRIS会自动使用系统配置的HugePage, 这会提高IRIS的内存使用效率。在线文档有这么一句： **在超大页内存中可用的内存数量应大于待分配的共享内存总数;否则，将不会使用超大页内存**。 因为 内存数据的总数是为Global, Routine, Lock等分配的内存的和。可以简单的从IRIS启动的日志message.log里看到。

 比如下面IRIS被分配了8913M的总内存：

```sh
12/24/21-16:28:58:490 (14080) 0 [Generic.Event] Allocated 8913MB shared memory: 8003MB global buffers, 300MB routine buffers
```

另外，因为操作系统自身的进程不会使用HugePage内存， 您也不能把内存都设置成HugePage。所以设置为**比IRIS需要的多一点就好**。

我们来看看配置HugePage的一个简单例子

```bash
# 查看服务器物理内存
[root@hmsCentOS7 ~]# cat /proc/meminfo | grep MemTotal
MemTotal:         995676 kB

#查看HugePage配置
[root@hmsCentOS7 ~]# cat /proc/meminfo | grep Huge
AnonHugePages:     67584 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB

# 修改配置，分配256个2M的HugePage
[root@hmsCentOS7 ~]# vi /etc/sysctl.conf
[root@hmsCentOS7 ~]# cat /etc/sysctl.conf | grep -v ^#
vm.nr_hugepages=256
#load新配置
[root@hmsCentOS7 ~]# sysctl -p
vm.nr_hugepages = 256
[root@hmsCentOS7 ~]

[root@hmsCentOS7 ~]# cat /proc/meminfo | grep Huge
AnonHugePages:      6144 kB
HugePages_Total:     256
HugePages_Free:      256
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
[root@hmsCentOS7 ~]#
```

如果此时您启动IRIS, 会出现“Allocated xxxxMB shared memory using Huge Pages"的日志记录，而且能在/proc/meminfo里看到HugePage的占用情况。

**禁用THP(Transparent Huge Pages)**

InterSystems建议在IRIS服务器禁用THP。RedHat或者CentOS是默认打开的：

```SH
[root@hmsCentOS7 ~]# cat /sys/kernel/mm/transparent_hugepage/enabled
[always] madvise never
[root@hmsCentOS7 ~]#
```

禁用THP您需要修改/etc/default/grub中的GRUB_CMDLINE_LINUX记录，添加“transparent_hugepage=never“

```sh
# 修改文件
[root@hmsCentOS7 ~]# cat /etc/default/grub | grep GRUB_CMDLINE
GRUB_CMDLINE_LINUX="crashkernel=auto spectre_v2=retpoline rd.lvm.lv=centos_hmscentos7/root rd.lvm.lv=centos_hmscentos7/swap rhgb quiet transparent_hugepage=never"

# regenerate grub2配置
[root@hmsCentOS7 ~]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-1160.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-1160.el7.x86_64.img
Found linux image: /boot/vmlinuz-0-rescue-bd06ad08438443f5a959cb7c0c264e89
Found initrd image: /boot/initramfs-0-rescue-bd06ad08438443f5a959cb7c0c264e89.img
done

#重启
[root@hmsCentOS7 ~]shutdown -r now

# 确认配置成功
[root@hmsCentOS7 ~]# cat /sys/kernel/mm/transparent_hugepage/enabled
always madvise [never]
[root@hmsCentOS7 ~]#
```

> [Redhat文档:配置THP](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/performance_tuning_guide/sect-red_hat_enterprise_linux-performance_tuning_guide-configuring_transparent_huge_pages)



**Dirty Page Cleanup**

在一个大内存(比如大于8G)的Linux系统中，以下参数的修改会提高大量flat-files的写性能，比如大量的文件拷贝或者IRIS backup。 

- dirty-background_ratio: InterSystems建议设置为5
- dirty_ratio: InterSystems建议设置为10

您需要修改/etc/sysctl.conf文件， 添加这两行：

```sh
vm.dirty_background_ratio=5
vm.dirty_ratio=10
```



###配置NTP服务

各个数据库服务之间，以及数据库服务器和应用服务，Web服务，打印服务等等之间，通常要求精准的时间同步。客户可以选择采用chornyd或者ntpd来配置NTP, 其中chornyd更新，也被这些年的Linux更为推荐。建议每个IRIS服务器配置至少2个NTP Server地址。 

有关配置NTP的方法请参考相关文档。

比如：[Redhat文档: CONFIGURING NTP USING THE CHRONY SUITE](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-configuring_ntp_using_the_chrony_suite#sect-differences_between_ntpd_and_chronyd) 



### 安装Web Server (optional)

IRIS提供Web服务需要单独安装的Web服务器。在Linux系统，IRIS支持Apache Web Server和Nginx。

生产环境中， 推荐将Web Server和IRIS安装在不同的硬件服务器上。如果不是分开安装，而是**要将两者安装在同一台服务器，那么最好最好先装好Web Server， 再安装IRIS。**

原因是: 在执行IRIS的安装脚本时，会提示用户是否需要连接那么先装Apache Web Server和Nginx是正确的顺序， 这样安装IRIS的时候可以将IRIS的Web服务组件，IRIS_WebGateway安装到Apache Web Server的安装目录，并生成相应的配置文件。 

```sh
# 修改httpd状态的常用命令
systemctl start httpd.service #启动apache
systemctl stop httpd.service #停止apache
systemctl restart httpd.service #重启apache
#设置开机启动
sudo systemctl enable httpd.service #设置开机启动
```

> 如果启动httpd显示错误“httpd: Could not reliably determine the server's fully qualified domain name”，您需要修改httpd.conf文件， 添加ServerName, 比如设为ServerName localhost:80



**到这里， 您已经基本完成了安装IRIS的准备工作，下面是IRIS的安装步骤。**



### 加载IRIS安装软件

```sh
# 解压安装文件

[root@hmsCentOS7 ~]# mkdir /tmp/iriskit
[root@hmsCentOS7 ~]# chmod og+rx /tmp/iriskit
[root@hmsCentOS7 ~]# umask 022
[root@hmsCentOS7 ~]# gunzip -c IRIS-2022.1.0.140.0-lnxrh7x64.tar.gz | (cd /tmp/iriskit; tar xf -)
[root@hmsCentOS7 iriskit]# cd /tmp/iriskit/IRIS-2022.1.0.140.0-lnxrh7x64/
[root@hmsCentOS7 IRIS-2022.1.0.140.0-lnxrh7x64]# ls
LICENSE  NOTICE  cplatname  dist  docs  irisinstall  irisinstall_client  irisinstall_silent  kitlist  lgpl.txt  package  tools
```

> tar -zxvf *.tar.gz是centos上常用的解压命令。上面使用的gunzip来着在线文档，据说是更通用的命令，适合无法用tar来操作.gz文件的linux环境。



###安装依赖的软件包

建议执行IRIS安装脚本前先检查需要的软件包。

```sh
# 查看安装需要的软件包
[root@hmsCentOS7 IRIS-2022.1.0.140.0-lnxrh7x64]# ./irisinstall --prechecker
Your system type is 'Red Hat Enterprise Linux (x64)'.
Checking Requirements ...
Requirement for openssl version 1.0.2 is satisfied.
Requirement for zlib version 1.2.7 is satisfied.
[root@hmsCentOS7 IRIS-2022.1.0.140.0-lnxrh7x64]#
```

除此以外，以下软件包是按情况可选的：

- snmpd: 如果您需要使用SNMP来监控IRIS
- jre或者jdk : 如果您需要IRIS服务器使用JDBC连接其他数据库
- unixODBC: 如果您需要IRIS服务器使用ODBC连接其他数据库
- policycoreutils-python, policycoreutils-devel, settroubleshoot-server : 如果您需要使用SELinux管理IRIS文件的读写。

它们只是IRIS工作时需要的软件包，IRIS的安装并不依赖或者检查它们。



###创建iris用户

安装iris需要root用户或者拥有root权限的sudoer。然而，按照Linux的常规做法，出于安全的考虑，IRIS的管理和运行应该有自己的用户和用户组。在安装过程中， 您会被要求提供以下的内容：

- **What user should be the owner of this instance?**
- **What group should be allowed to start and stop？**

"owner of instance"和“group to start and stop"可以是一个用户和它的用户组。比如您创建了一个irisowner的用户， 它默认的用户组irisowner可以作为“group start to stop"。当然您也可以专门创建一个用户组， 比如irisadmin, 将您的管理员tom, jerry, mickey等等加入irisadmin。

在标准的安装步骤下， 用户组irisowner和用户组irisadmin权限相同， 因此最简单的方式就是使用irisowner用户和irisowner用户组。

```sh
# 创建iris owner
useradd irisowner
# 创建您的管理员团队
[root@hmsCentOS7 mail]# useradd -N tom
[root@hmsCentOS7 mail]# useradd -N jerry
[root@hmsCentOS7 mail]# useradd -N iscbackup
[root@hmsCentOS7 mail]# gpasswd irisowner -a tom
Adding user tom to group irisowner
[root@hmsCentOS7 mail]# gpasswd irisowner -a jerry
Adding user jerry to group irisowner
[root@hmsCentOS7 mail]# gpasswd irisowner -a iscbackup
Adding user iscbackup to group irisowner
[root@hmsCentOS7 mail]# cat /etc/group | grep irisowner
irisowner:x:1000:tom,jerry,iscbackup
[root@hmsCentOS7 mail]#
```

*如果需要，您可以把tom, jerry等用户加入sudoer列表。*

**实际上， 在后面安装脚本的执行过程中， 还有2个重要的用户被创建出来, 他们是：**

- **irisusr**: 所有iris进程使用这个user运行。它还用来访问数据库和journal。在一个安全的系统里，irisusr组里就只有一个用户irisusr, 也不要添加其他的user。
- **iscagent**: 用于运行镜像使用的iscagent进程

IRIS用户和管理员不要动这两个用户。



###执行IRIS安装脚本

安装脚本会设置user, group,和安装后文件的权限，您需要在安装前检查umask， 确认设置为022。

> If an instance with this name already exists, the program asks if you wish to upgrade it

**执行标准的安装流程**

```bash
# 开始安装步骤

[root@hmsCentOS7 iriskit]# cd /tmp/iriskit/IRIS-2022.1.0.140.0-lnxrh7x64/
[root@hmsCentOS7 IRIS-2022.1.0.140.0-lnxrh7x64]# ls
LICENSE  NOTICE  cplatname  dist  docs  irisinstall  irisinstall_client  irisinstall_silent  kitlist  lgpl.txt  package  tools
[root@hmsCentOS7 IRIS-2022.1.0.140.0-lnxrh7x64]# ./irisinstall

Your system type is 'Red Hat Enterprise Linux (x64)'.
Currently defined instances:

Enter instance name: iris
Do you want to create InterSystems IRIS instance 'iris' <Yes>?

Enter a destination directory for the new instance.
Directory: /isc/iris

------------------------------------------------------------------
NOTE: Users should not attempt to access InterSystems IRIS while
      the installation is in progress.
------------------------------------------------------------------

Select installation type.
    1) Development - Install InterSystems IRIS server and all language bindings
    2) Server only - Install InterSystems IRIS server
    3) Custom
Setup type <1>? 1

How restrictive do you want the initial Security settings to be?
"Minimal" is the least restrictive, "Locked Down" is the most secure.
    1) Minimal
    2) Normal
    3) Locked Down
Initial Security settings <1>? 2

What user should be the owner of this instance? irisowner
An InterSystems IRIS account will also be created for user irisowner.

Install will create the following InterSystems IRIS accounts for you:
_SYSTEM, Admin, SuperUser, irisowner and CSPSystem.
Please enter the common password for _SYSTEM, Admin, SuperUser and irisowner:
Re-enter the password to confirm it:

Please enter the password for CSPSystem:
Re-enter the password to confirm it:

What group should be allowed to start and stop
  this instance? irisowner

Do you want to install IRIS Unicode support <Yes>?

InterSystems IRIS did not detect a license key file

Do you want to enter a license key <No>?

Please review the installation options:
------------------------------------------------------------------
Instance name: iris
Destination directory: /isc/iris
InterSystems IRIS version to install: 2022.1.0.140.0
Installation type: Development
Unicode support: Y
Initial Security settings: Normal
User who owns instance: irisowner
Group allowed to start and stop instance: irisowner
Effective group for InterSystems IRIS processes: irisusr
Effective user for InterSystems IRIS SuperServer: irisusr
SuperServer port: 1972
WebServer port: 52773
JDBC Gateway port: 53773
Web Gateway: using built-in web server
Not installing IntegratedML
------------------------------------------------------------------

Confirm InterSystems IRIS installation <Yes>?

Starting installation
Starting up InterSystems IRIS for loading...
../bin/irisinstall -s . -B -c c -C /isc/iris/iris.cpf*iris -W 1 -g2
Starting Control Process
Allocated 238MB shared memory
32MB global buffers, 80MB routine buffers
Creating a WIJ file to hold 32 megabytes of data
IRIS startup successful.
locale: Cannot set LC_CTYPE to default locale: No such file or directory
locale: Cannot set LC_ALL to default locale: No such file or directory
System locale setting is 'en_US.UTF-8'
This copy of InterSystems IRIS has been licensed for use exclusively by:
Local license key file not found.
Copyright (c) 1986-2022 by InterSystems Corporation
Any other use is a violation of your license agreement

^^/isc/iris/mgr/>

^^/isc/iris/mgr/>
Start of IRIS initialization
Loading system routines
Updating system TEMP and LOCALDATA databases
Installing National Language support

Setting IRISTEMP default collation to IRIS standard (5)
Loading system classes
Updating Security database
Loading system source code
Building system indices
Updating Audit database
Updating Journal directory
Updating User database
Updating Interoperability databases
Scheduling inventory scan
IRIS initialization complete

See the iboot.log file for a record of the installation.

Starting up InterSystems IRIS...
Once this completes, users may access InterSystems IRIS
Starting IRIS
Using 'iris.cpf' configuration file

Starting Control Process
Global buffer setting requires attention.  Auto-selected 25% of total memory.
Allocated 463MB shared memory
243MB global buffers, 80MB routine buffers
Creating a WIJ file to hold 99 megabytes of data
This copy of InterSystems IRIS has been licensed for use exclusively by:
Local license key file not found.
Copyright (c) 1986-2022 by InterSystems Corporation
Any other use is a violation of your license agreement

1 alert(s) during startup. See messages.log for details.

You can point your browser to http://hmsCentOS7:52773/csp/sys/UtilHome.csp
to access the management portal.

Installation completed successfully
[root@hmsCentOS7 IRIS-2022.1.0.140.0-lnxrh7x64]#
```



**执行customer安装IRIS+WebGateway**

如果要在本机安装WebGateway, 需要在执行安装脚本的过程中，当被问到：“Select installation type.”时选择Customer, 而不是Development。这样后面就脚本将在Web Server上加载IRIS专用的动态连接库，并创建和Web Server连接的IRIS Web网关。

在此之前， 您还需要配置安装semanage来保证可以配置SELinux。安装脚本会检查，如果没有发现semanag, 会提示您停止继续安装。 

```sh
# 安装semanage, 确认命令可以工作。省略输出
[root@hmsCentOS7 ~]# yum install policycoreutils-devel
[root@hmsCentOS7 ~]# yum instll setroubleshoot-server
[root@hmsCentOS7 ~]# yum install setroubleshoot-server
[root@hmsCentOS7 ~]# semanage -h
...
[root@hmsCentOS7 ~]# 
```

假设你已经创建了一个/isc/iris2的目录，然后就可以安装了。注意下面使用的superserver和web端口是51773和52774。这是默认的第2个IRIS实例的端口号，您也可以输入其他端口号或者在安装后在维护页面修改这些端口。 

```sh
[root@hmsCentOS7 IRIS-2022.1.0.140.0-lnxrh7x64]# ./irisinstall

Your system type is 'Red Hat Enterprise Linux (x64)'.

Currently defined instances:

IRIS instance 'IRIS'
	directory:    /isc/iris
	versionid:    2022.1.0.140.0
	datadir:      /isc/iris
	conf file:    iris.cpf  (SuperServer port = 1972, WebServer = 52773)
	status:       down, last used Fri Apr  8 13:32:47 2022
	product:      InterSystems IRIS


Enter instance name: iris2
Do you want to create InterSystems IRIS instance 'iris2' <Yes>?

Enter a destination directory for the new instance.
Directory: /isc/iris2

------------------------------------------------------------------
NOTE: Users should not attempt to access InterSystems IRIS while
      the installation is in progress.
------------------------------------------------------------------

Select installation type.
    1) Development - Install InterSystems IRIS server and all language bindings
    2) Server only - Install InterSystems IRIS server
    3) Custom
Setup type <1>? 3

How restrictive do you want the initial Security settings to be?
"Minimal" is the least restrictive, "Locked Down" is the most secure.
    1) Minimal
    2) Normal
    3) Locked Down
Initial Security settings <1>? 2

What user should be the owner of this instance? irisowner
An InterSystems IRIS account will also be created for user irisowner.


Install will create the following InterSystems IRIS accounts for you:
_SYSTEM, Admin, SuperUser, irisowner and CSPSystem.
Please enter the common password for _SYSTEM, Admin, SuperUser and irisowner:
Re-enter the password to confirm it:


Please enter the password for CSPSystem:
Re-enter the password to confirm it:

What group should be allowed to start and stop
  this instance? irisowner

Do you want to configure additional security options <No>?

Do you want to install IRIS Unicode support <Yes>?

Enter the SuperServer port number <51773>:

Enter the WebServer port number <52774>:

Do you want to configure the Web Gateway to use an existing web server <No>? yes

Specify the WebServer type. Choose "None" if you want to configure
your WebServer manually.
    1) Apache
    2) None
WebServer type <2>? 1

Enter user name used by Apache server to run its worker processes <apache>:

Please enter location of Apache configuration file [/etc/httpd/conf/httpd.conf]:
// /etc/httpd/conf.d/isc.conf
Please enter location of Apache executable file </usr/sbin/httpd>:
Apache version 2.4 is detected.

Apache httpd server will be restarted during the install.

Please enter destination directory for Web Gateway files [/opt/webgateway]:

InterSystems IRIS did not detect a license key file

Do you want to enter a license key <No>?

Do you want to install IntegratedML (this feature requires additional 1166MB of disk space) <No>?

Please review the installation options:
------------------------------------------------------------------
Instance name: iris2
Destination directory: /isc/iris2
InterSystems IRIS version to install: 2022.1.0.140.0
Installation type: Custom
Unicode support: Y
Initial Security settings: Normal
User who owns instance: irisowner
Group allowed to start and stop instance: irisowner
Effective group for InterSystems IRIS processes: irisusr
Effective user for InterSystems IRIS SuperServer: irisusr
SuperServer port: 51773
WebServer port: 52774
JDBC Gateway port: 53773
Web Gateway: installed into /opt/webgateway
Apache web server will be configured for Web Gateway
Not installing IntegratedML
------------------------------------------------------------------

Confirm InterSystems IRIS installation <Yes>? yes

Starting installation
    Updating Apache configuration file ...
    - /etc/httpd/conf/httpd.conf

Starting up InterSystems IRIS for loading...
../bin/irisinstall -s . -B -c c -C /isc/iris2/iris.cpf*iris2 -W 1 -g2
Starting Control Process
Allocated 238MB shared memory
32MB global buffers, 80MB routine buffers
Creating a WIJ file to hold 32 megabytes of data
IRIS startup successful.
locale: Cannot set LC_CTYPE to default locale: No such file or directory
locale: Cannot set LC_ALL to default locale: No such file or directory
System locale setting is 'en_US.UTF-8'
This copy of InterSystems IRIS has been licensed for use exclusively by:
Local license key file not found.
Copyright (c) 1986-2022 by InterSystems Corporation
Any other use is a violation of your license agreement

^^/isc/iris2/mgr/>

^^/isc/iris2/mgr/>
Start of IRIS initialization
Loading system routines
Updating system TEMP and LOCALDATA databases
Installing National Language support

Setting IRISTEMP default collation to IRIS standard (5)
Loading system classes
Updating Security database
Loading system source code
Building system indices
Updating Audit database
Updating Journal directory
Updating User database
Updating Interoperability databases
Scheduling inventory scan
IRIS initialization complete

See the iboot.log file for a record of the installation.

Starting up InterSystems IRIS...
Once this completes, users may access InterSystems IRIS
Starting IRIS2
Using 'iris.cpf' configuration file

Starting Control Process
Global buffer setting requires attention.  Auto-selected 25% of total memory.
Allocated 463MB shared memory
243MB global buffers, 80MB routine buffers
Creating a WIJ file to hold 99 megabytes of data
This copy of InterSystems IRIS has been licensed for use exclusively by:
Local license key file not found.
Copyright (c) 1986-2022 by InterSystems Corporation
Any other use is a violation of your license agreement

1 alert(s) during startup. See messages.log for details.


You can point your browser to http://hmsCentOS7:52774/csp/sys/UtilHome.csp
to access the management portal.

Installation completed successfully
[root@hmsCentOS7 IRIS-2022.1.0.140.0-lnxrh7x64]#
```



上面步骤中当提示：”Please enter location of Apache configuration file [/etc/httpd/conf/httpd.conf]: “ 的时候我使用了默认选项/etc/httpd/conf/httpd.conf, IRIS安装脚本会修改httpd.conf文件， 加入连接IRIS的配置。这里更好的做法是先在/etc/httpd/conf.d里创建一个单独的配置文件，比如isc.conf， 在回答上面的提示是输入**/etc/httpd/conf.d/isc.conf**。很遗憾您需要自己手工先创建这个文件，安装脚本不会替您创建它。

### 安装后的检查

**检查安装日志**

查看上面的安装日志，确认没有错误发生。

**查看新创建的用户irisusr和iscagent**

```sh
[root@hmsCentOS7 ~]# id irisusr
uid=1001(irisusr) gid=1001(irisusr) groups=1001(irisusr)
[root@hmsCentOS7 ~]# id iscagent
uid=1002(iscagent) gid=1002(iscagent) groups=1002(iscagent)
[root@hmsCentOS7 ~]#
```

**查看文件系统权限**

```sh
[root@hmsCentOS7 ~]# ls -l /isc
total 8
drwxr-xr-x.  2 root      root       6 Apr  1 23:57 data
drwxrwxr-x. 14 irisowner irisusr 4096 Apr  6 14:28 iris
drwxr-xr-x.  3 root      root    1024 Apr  1 23:57 jrnpri
drwxr-xr-x.  3 root      root    1024 Apr  1 23:57 jrnsec
[root@hmsCentOS7 ~]# ls -l /isc/iris
total 112
-r--r--r--.  1 irisowner irisusr   11358 Apr  6 14:28 LICENSE
-r--r--r--.  1 irisowner irisusr     534 Apr  6 14:28 NOTICE
-rwxr-xr-x.  1 irisowner irisusr    4497 Mar  7 15:20 ODBCinstall
drwxrwxr-x.  2 irisowner irisusr      26 Apr  6 14:27 SNMP
-rwxrw-r--.  1 irisusr   irisusr    9703 Apr  6 14:28 _LastGood_.cpf
drwxr-xr-x.  2 root      irisowner  8192 Apr  6 14:28 bin
drwxrwxr-x.  7 irisowner irisusr     106 Apr  6 14:27 csp
drwxr-xr-x. 17 irisowner irisusr     214 Apr  6 14:28 dev
drwxrwxr-x.  3 irisowner irisusr      20 Apr  6 14:28 devuser
drwxr-xr-x.  3 root      root         21 Apr  6 14:28 dist
drwxr-xr-x.  5 irisowner irisusr     156 Apr  6 14:28 docs
drwxr-xr-x.  5 irisowner irisusr      68 Apr  6 14:28 fop
drwxrwxr-x.  5 irisowner irisusr      41 Apr  6 14:27 httpd
-rw-rw-r--.  1 root      root       9557 Apr  6 14:28 install.cpf
-rw-rw-r--.  1 irisowner irisusr    9732 Apr  6 14:28 iris.cpf
-rwxr-xr-x.  1 irisowner irisusr    9557 Apr  6 14:28 iris.cpf_20220406
-r-xr-x---.  4 irisowner irisowner   909 Apr  6 14:28 irisforce
-r-xr-x---.  4 irisowner irisowner   909 Apr  6 14:28 irisstart
-r-xr-x---.  4 irisowner irisowner   909 Apr  6 14:28 irisstop
-r--r--r--.  1 irisowner irisusr    7639 Apr  6 14:28 lgpl.txt
drwxr-xr-x.  6 irisowner irisusr      80 Apr  6 14:28 lib
drwxrwxr-x. 13 irisowner irisusr    4096 Apr  6 14:28 mgr
-rw-------.  1 root      root       3299 Apr  6 14:28 parameters.isc
drwxr-xr-x.  2 irisowner irisusr     237 Apr  6 14:27 patrol
[root@hmsCentOS7 ~]# ls -l /isc/iris/mgr
total 163952
-rw-r-----. 1 irisowner irisusr  62914560 Apr  6 14:31 IRIS.DAT
-rw-rw----. 1 irisowner irisusr 104857600 Apr  6 14:36 IRIS.WIJ
drwxr-xr-x. 2 irisowner irisusr      4096 Apr  6 14:28 Locale
-rwxrw-r--. 1 irisusr   irisusr        66 Apr  6 14:28 SystemMonitor.log
drwxrwxr-x. 2 irisusr   irisusr         6 Apr  6 14:28 Temp
-rwxrw-r--. 1 irisusr   irisusr       134 Apr  6 14:28 alerts.log
-rwxrw-r--. 1 irisusr   irisusr     13759 Apr  6 14:28 ensinstall.log
drwxrwxr-x. 3 irisowner irisusr        36 Apr  6 14:28 enslib
-rwxr-xr-x. 1 irisowner irisusr     38319 Apr  6 14:28 iboot.log
-rwxrwxrwx. 1 irisowner irisusr         0 Apr  6 14:28 ilock
-rw-rw----. 1 irisowner irisusr        10 Apr  6 14:28 iris.ids
-rw-rw----. 1 irisowner irisusr        30 Apr  6 14:28 iris.lck
-rw-rw-rw-. 1 irisowner irisusr         2 Apr  6 14:28 iris.shid
-rw-rw-r--. 1 irisowner irisusr         5 Apr  6 14:28 iris.use
drwxrwxr-x. 3 irisowner irisusr        52 Apr  6 14:28 irisaudit
drwxrwxr-x. 3 irisowner irisusr        36 Apr  6 14:28 irislib
drwxrwxr-x. 3 irisowner irisusr        52 Apr  6 14:28 irislocaldata
-rw-rw----. 1 irisowner irisusr       933 Apr  6 14:28 irisodbc.ini
drwxrwxr-x. 3 irisowner irisusr        52 Apr  6 14:28 iristemp
drwxrwxr-x. 2 irisowner irisusr        42 Apr  6 14:28 journal
-rw-rw----. 1 irisowner irisusr       211 Apr  6 14:28 journal.log
-rw-rw-r--. 1 irisowner irisusr     14146 Apr  6 14:28 messages.log
drwxrwxr-x. 2 irisowner irisusr         6 Apr  6 14:27 python
-rw-rw-rw-. 1 irisowner irisusr        55 Apr  6 14:28 startup.last
drwxrwxr-x. 2 irisowner irisusr         6 Apr  6 14:28 stream
drwxrwxr-x. 3 irisowner irisusr        52 Apr  6 14:28 user
[root@hmsCentOS7 ~]#
```

**修改文件系统权限**

将/isc/data, /isc/jrnpri, /isc/jrnsec文件系统的user和group，以及权限修改到下面的样子

```sh
[root@hmsCentOS7 ~]# chown irisowner:irisusr /isc/data /isc/jrnpri /isc/jrnsec
[root@hmsCentOS7 ~]# chmod 775 /isc/data /isc/jrnpri /isc/jrnsec
[root@hmsCentOS7 ~]# ls -l /isc
total 8
drwxrwxr-x.  2 irisowner irisusr    6 Apr  1 23:57 data
drwxrwxr-x. 14 irisowner irisusr 4096 Apr  6 14:28 iris
drwxrwxr-x.  3 irisowner irisusr 1024 Apr  1 23:57 jrnpri
drwxrwxr-x.  3 irisowner irisusr 1024 Apr  1 23:57 jrnsec
[root@hmsCentOS7 ~]#
```

**打开防火墙IRIS的TCP端口**

IRIS系统使用以下TCP端口: 

- superserver port: 默认是1972
- web port: 默认是52773
- mirror port: 默认是2188
- license server port: 默认是4001, （很少使用）

```sh
[root@hmsCentOS7 iris]# firewall-cmd --permanent --add-port=1972/tcp
success
[root@hmsCentOS7 iris]# firewall-cmd --permanent --add-port=52773/tcp
success
[root@hmsCentOS7 iris]# firewall-cmd --permanent --add-port=2188/tcp
success
[root@hmsCentOS7 iris]# firewall-cmd --list-port
1972/tcp 2188/tcp 52773/tcp 
[root@hmsCentOS7 iris]# firewall-cmd --reload
success
[root@hmsCentOS7 iris]#
```

**访问系统管理界面**

通过SMP(System Management Portal)的URL,  您可以访问IRIS。

http://host-ipaddr:52773/csp/sys/UtilHome.csp

用户名使用superuser, 密码为您安装创建的密码。

**激活IRIS License**

到管理界面的“系统 > 软件许可颁发 > 软件许可授权码”，加载在本机保存的IRIS许可。

注意弹出窗口的文件选项默认是.key文件。所以您上传文件到Linux服务器前最好把文件改为iris.key。另外，文件和文件夹一定要可以被irisowner访问。如果放在/root文件夹，或者不允许other用户访问，您要么在选文件的窗口无法找到这个文件，要么无法成功激活。 



**创建数据库**

到 "系统 > 配置 > 本地数据库"创建数据库。查看在/isc/data里的文件夹和权限。举例：创建demo数据库到/isc/data/demo:

```sh
[root@hmsCentOS7 isc]# ls -Rl /isc/data
/isc/data:
total 0
drwxrwxr-x. 3 irisusr irisusr 52 Apr  6 15:37 demo

/isc/data/demo:
total 1028
-rw-rw----. 1 irisusr   irisusr 1048576 Apr  6 15:38 IRIS.DAT
-rw-rw----. 1 irisowner irisusr      30 Apr  6 15:37 iris.lck
drwxrwxr-x. 2 irisusr   irisusr       6 Apr  6 15:37 stream

/isc/data/demo/stream:
total 0
[root@hmsCentOS7 isc]#
```

**设置Journal文件夹**

到“系统 > 配置 > Journal 设置”设置journal的主备路径到/isc/jrnprm和/isc/jrnsec。

查看两个路径里的文件权限是660(rw-rw----)

```sh
[root@hmsCentOS7 opt]# ls -l /isc/jrnpri
total 272
-rw-rw----. 1 irisowner irisusr 262144 Apr  6 15:29 20220406.002
-rw-rw----. 1 irisowner irisusr     30 Apr  6 15:29 iris.lck
drwx------. 2 irisowner irisusr  12288 Apr  1 23:57 lost+found
[root@hmsCentOS7 opt]# ls -l /isc/jrnsec
total 15
-rw-rw----. 1 irisowner irisusr    30 Apr  6 15:29 iris.lck
drwx------. 2 irisowner irisusr 12288 Apr  1 23:57 lost+found
[root@hmsCentOS7 opt]#
```

**启动ISCAgent**

如果需要配置镜像，服务器需要人工启动ISCAgent

 ```bash
# 启动iscagent服务
systemctl start ISCAgent.service

# 设置iscagent
systemctl enable ISCAgent.service
 ```



**SELinux配置**(Optional)

如果您上面执行的是“**执行customer安装以安装WebGateway**”，那么这里您需要配置SELinux。

SELinux会限制httpd对系统其它文件的访问权限，比如说//usr/sbin/httpd无法访问IRIS的WebGateway, 它工作在/opt/webgateway/bin目录。你需要做以下几步来解除这个限制：

- 允许httpd读写WebGateway文件

```sh
# httpd_sys_content_t – set on directories that Apache is allowed access to,
# httpd_sys_rw_content_t – set on directories that Apache is allowed read/write access # httpd_sys_script_exec_t – used for directories that contain executable scripts.

[root@hmsCentOS7 conf]# ls -Z /opt/webgateway/conf
-rw-------. apache root unconfined_u:object_r:httpd_sys_content_t:s0 CSP.ini
-rw-------. apache root unconfined_u:object_r:httpd_sys_content_t:s0 CSPRT.ini
[root@hmsCentOS7 conf]# semanage fcontext -a -t httpd_sys_rw_content_t /opt/webgateway/conf/CSP.ini
[root@hmsCentOS7 conf]# restorecon -v /opt/webgateway/conf/CSP.ini
restorecon reset /opt/webgateway/conf/CSP.ini context unconfined_u:object_r:httpd_sys_content_t:s0->unconfined_u:object_r:httpd_sys_rw_content_t:s0
[root@hmsCentOS7 conf]# semanage fcontext -a -t httpd_sys_rw_content_t /opt/webgateway/conf/CSPRT.ini
[root@hmsCentOS7 conf]# restorecon -v /opt/webgateway/conf/CSPRT.ini
restorecon reset /opt/webgateway/conf/CSPRT.ini context unconfined_u:object_r:httpd_sys_content_t:s0->unconfined_u:object_r:httpd_sys_rw_content_t:s0
[root@hmsCentOS7 conf]# ls -Z
-rw-------. apache root unconfined_u:object_r:httpd_sys_rw_content_t:s0 CSP.ini
-rw-------. apache root unconfined_u:object_r:httpd_sys_rw_content_t:s0 CSPRT.ini
[root@hmsCentOS7 conf]#

[root@hmsCentOS7 webgateway]# semanage fcontext -a -t httpd_sys_rw_content_t /opt/webgateway/logs/CSP.log
[root@hmsCentOS7 webgateway]# restorecon -v /opt/webgateway/logs/CSP.log
restorecon reset /opt/webgateway/logs/CSP.log context unconfined_u:object_r:httpd_sys_content_t:s0->unconfined_u:object_r:httpd_sys_rw_content_t:s0
[root@hmsCentOS7 webgateway]# 
```

- 允许httpd读写IRIS的Web Application文件，假设文件目录是/isc/web, 命令中的“-R”是recursively，递归执行。

```sh
semanage fcontext -a -t httpd_sys_rw_content_t "/isc/web(/.*)?" 
restorecon -Rv /isc/web/
```

- 运行httpd到IRIS的连接, （最新版本也许不需要手工配置了）

```sh
# 默认的superserver端口为1972, 上面的iris2使用的是51773

[root@hmsCentOS7 ~]# semanage port -a -t http_port_t -p tcp 51773
ValueError: Port tcp/51773 already defined
[root@hmsCentOS7 ~]# semanage port -a -t http_port_t -p tcp 1972
[root@hmsCentOS7 ~]# semanage port -l | grep http_port
http_port_t                    tcp      1972, 51773, 80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
[root@hmsCentOS7 ~]# setsebool -P httpd_can_network_connect 1
[root@hmsCentOS7 ~]#

```



到这里， 您的IRIS安装过程已经结束了。后面的实施工作可能包括：

- IRIS配置
- 镜像配置
- WebGateway配置

我会在其他文章中介绍上述内容。



另外， 请注意**重启(reboot)在安装过程中十分重要**： 通常的情况是，至少有一些配置在第一次重启时并不存在。这可能包括被挂载的卷、/etc/fstab中的错误、启动时不启动的服务、在没有重载服务的情况下进行的配置修改等等。

一旦你认为服务器已经完成，并验证所有的配置都符合预期，而且应用程序仍在工作，那么重新启动服务器是非常明智的。如果不这样做，也不测试集群（例如，一次重启一个节点），可能意味着在第一次重启或HA事件中出现灾难。

*The End*

