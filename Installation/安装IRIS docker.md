# IRIS Docker的安装

IRIS相比Caché在部署上的一个进步是支持docker。即便不是云部署， 使用docker也带来非常多的便利。 尤其是在开发测试环节，由于docker的使用更便捷，除非要模拟客户的环境或者做规定的性能测试，我在测试中基本已经不再使用本机的实例或者虚机。IRIS的联机文档有[详细的IRIS docker安装使用指导](https://irisdocs.intersystems.com/irislatest/csp/docbook/Doc.View.cls?KEY=ADOCK#ADOCK_iris_running_compose)，本文只是一个简单的，快速上手的**在测试环境**安装IRIS docker的简单步骤，尤其适合初学者。

注意Windows上docker可能会遇到这样那样的问题，因此通常还是推荐在Linux或者Mac OS上使用。正式的生产环境的IRIS docker container也是不支持Windows系统的。 

> Referrence
- [First Look: InterSystems Products in Docker Containers](https://irisdocs.intersystems.com/irislatest/csp/docbook/Doc.View.cls?KEY=AFL_containers)
- [Running an InterSystems IRIS Container: Docker Run ExamplesRunning an InterSystems IRIS Container: Docker Run Examples](https://irisdocs.intersystems.com/irislatest/csp/docbook/Doc.View.cls?KEY=ADOCK#ADOCK_iris_running_dockerrun)



###在操作系统上安装Docker环境

[Docker官方文档的安装步骤](https://docs.docker.com/engine/install/)非常清晰，我按照上面的步骤在MAC和CentOS上安装docker从来没有出现过问题。恰恰是这个文档中没有的Redhat上的安装，过程中曾碰到过几个问题，在网上搜索了答案，也不算困难。

简单的复述一下步骤：root用户的最简单安装，没有权限问题， 没有docker网络修改等等：

1. 安装yum-util, 用于添加yum源,如果您的系统中已经有了yum-utils包这步可以省略
  
   ```sh
    sudo yum install -y yum-utils
   ```
2. 添加docker的Repo到yum源并确认
  
   ```sh
    sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    sudo yum repolist
   ```
3. 安装3个docker组件
  
   ```sh
    sudo yum install docker-ce docker-ce-cli containerd.io
   ```
4. 启动Docker
  
   ```sh
    sudo systemctl start docker
   ```
5. 确认安装成功。'docker run'命令会去本地的库里找'hello-world'，没有找到就去网上下一个"images",然后创建一个docker容器给你使用。

   ```sh
    sudo docker run hello-world
   ```


安装结束您想要到<https://hub.docker.com/>注册一个账户，用来下载上传docker image。 我的电脑上就装了nginx, mariadb, redis等等各种各样的docker image. 



下面是一点实例

**docker login**   

```sh
UKSBK2HTAYLOR:~ hma$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: imess
Password:
Login Succeeded
UKSBK2HTAYLOR:~ hma$
```

**docker pull image**

```sh
bogon:~ hma$ docker pull 
Using default tag: latest
latest: Pulling from library/hello-world
b8dfde127a29: Pull complete
Digest: sha256:308866a43596e83578c7dfa15e27a73011bdd402185a84c5cd7f32a88b501a24
Status: Downloaded newer image for hello-world:latest
docker.io/library/hello-world:latest
bogon:~ hma$
```

> docker default network change in CentOS: 

## 在那里下载IRIS Container

在线文档[Container Images Available from InterSystems](https://docs.intersystems.com/components/csp/docbook/Doc.View.cls?KEY=PAGE_containerregistry#PAGE_containerregistry)中介绍了<https://containers.intersystems.com>网站上可以下载的IRIS images的列表，其中community版本不需要license，其他的版本需要从InterSystems处获得IRIS docker版的专用license. 

下面是登录并下载iris docker image的记录。从网页登录<https://containers.intersystems.com>，您会得到docker的登录密码"使用的密码"jaRWSBJjcUcNprCKTuMX10PYHNq2IYPrAQoYdp6Siokb"。

```sh
[root@centos7 ~]# docker login -u="hma" -p="jaRWSBJjcUcNprCKTuMX10PYHNq2IYPrAQoYdp6Siokb" containers.intersystems.com
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[root@centos7 ~]# docker pull containers.intersystems.com/intersystems/iris-ml:2020.3.0.304.0
2020.3.0.304.0: Pulling from intersystems/iris-ml
5c939e3a4d10: Pull complete
c63719cdbe7a: Pull complete
19a861ea6baf: Pull complete
651c9d2d6c4f: Pull complete
d21839215a64: Pull complete
7995f836674b: Pull complete
841ee3aaa7aa: Pull complete
739c318c2223: Pull complete
d76886412dda: Pull complete
Digest: sha256:4c62690f4d0391d801d3ac157dc4abbf47ab3d8f6b46144288e0234e68f8f10e
Status: Downloaded newer image for containers.intersystems.com/intersystems/iris-ml:2020.3.0.304.0
containers.intersystems.com/intersystems/iris-ml:2020.3.0.304.0
[root@centos7 ~]#
```

另外，您也可以从InterSystems的技术支持网站下载iris docker的压缩包"iris*.tar.gz"，然后使用"docker load"安装， 比如：

```sh
CNMBPHMA:~ hma$ docker load -i iris-2020.4.0.524.0-docker.tar.gz
```


## 创建并运行IRIS Container

###简单的办法

如果是只是简单的测试，可以直接运行，但如果不是community版本的docker, 您还需要把license拷贝到container内部。 比如下面的步骤

```sh
CNMBPHMA:~ hma$ docker run -d -i -p 52773:52773 -p 51773:51773 --name iris20204 intersystems/irishealth:2020.2.0.211.0
CNMBPHMA:~ hma$ docker cp iris.key iris20204:/usr/irissys/mgr/iris.key
```

需要注意的是，不同版本的超级端口可能不同，有些是1972，有些是51773。为了省事，您也许可以使用"-P"把所有container内部的端口都映射出来。 

除了要拷贝license到container内部，在做测试的时候，您可能还要经常的使用"docker cp"把各种测试文件拷入container。而且，当查看和修改container内部文件的使用，还需要使用下面的命令进入container内部：

```sh
CNMBPHMA:~ hma$ docker exec -it iris20204 /bin/sh
```

因此，如果是您需要一个经常使用的iris container环境，我还是建议您使用下面的方法。

### IRIS Container使用外部存储





简单的说，下面的命令

- 使用"--volume"参数为container创建外部存储，将host上的"/root/data/dur"文件夹和container内部的"/dur"文件夹链接起来。(**/root/data/dur文件夹要在host上手工创建**，这只是一个示意，您可以使用任何可用的目录)

- ISC_DATA_DIRECTORY环境设置会在创建iris container时在/dur文件夹下创建一个子目录”/dur/irisepy"，也就是在主机的"/root/data/dur"目录下创建一个子目录”/root/data/dur/irisepy"。所有IRIS运行的日志，客户配置，mgr目录下的用户数据都会保存在这个目录。

    因为"/dur"是外部存储，即使docker container被删除数据也不会丢失。这对container的重新创建或者iris升级都带来很大的方便。在线文档上对如果覆盖或者升级带有外部存储的iris container有详细的介绍。 

- “key"参数定义了在IRIS container创建时会把"/dur/license"目录下的iris.key拷贝到iris安装目录的mgr下并激活。也就是说，在运行命令前， **您需要在host上在"/root/data/dur"目录下创建"license"子目录并将iris.key拷贝进去**。

    ```bash
    //create irisepy
    docker run --name irisepy --init -d -p 10203:1972 -p 10204:52773 -v /Users/hma/InterSystems/external/:/dur --env ISC_DATA_DIRECTORY=/dur/irisepy containers.intersystems.com/intersystems/iris-ml:2020.3.0.304.0 --key /dur/license/iris.key
    
    //create irisml
    docker run --name irisml --init -d -p 10203:1972 -p 10204:52773 -v /Users/hma/InterSystems/external/:/dur --env ISC_DATA_DIRECTORY=/dur/irisml containers.intersystems.com/intersystems/irishealth-ml:2021.2.0.617.0 --key /dur/license/iris.key         
             
    ```
    

 希望您喜欢使用docker

## 附录一: 简单的docker命令

**images操作**

    docker load -i ...

docker tag docker.intersystems.com/intersystems/iris:2018.1.1.888.0 acme/iris:stable

```sh
docker run -d -i   
docker run -d -i -p 52773:52773 -p 51773:51773 --name iris20194 intersystems/iris:2019.4.0.379.0  
docker cp iris.key   
CNMBPHMA:SEkeys hma$ docker exec -it 793befcc996c /bin/bash  
irisowner@793befcc996c:~$   
```

注意不通的操作系统命令格式不一样，可能是/bin/sh
在MAC上两个都可以， 但进去用户是irisowner, 不是root

# 附录（有关docker安装，命令的cheat sheet)


**docker创建和运行**


**docker container管理**


```sh
docker ps                           # 查看正在运行的container  或者 docker container ls   
docker ps -a                        # 查看所有已经创建的container
# container的stop, start, restart
# container修改标签 
docker rm dockerName            # 删除container
docker stop `docker ps -q`  		# 停止所有container
docker rm `docker ps -aq`   		# 删除所有container
```





## 附录2 ：安装docker的排错(2021-03-29 笔记)

两个redhat的docker不工作。

Step1. 收拾yum, 使用yum重新安装了一下

step 2: Permission Denied Error


post-install里面要设置用户加入docker权限

<https://linuxhandbook.com/docker-permission-denied/>


```sh
    sudo groupadd docker
    sudo usermod -aG docker $USER

    Verify that your user has been added to docker group by listing the users of the group. You probably have to log out and log in back again.

    abhishek@itsfoss:~$ groups
    abhishek adm cdrom sudo dip plugdev lpadmin sambashare docker
    If you check your groups and docker groups is not listed even after logging out, you may have to restart Ubuntu. To avoid that, you can use the newgrp command liks this:

    newgrp docker
```


```sh
    Further troubleshooting
    In some cases, you may need to add additional permissions to some files specially if you have run the docker commands with sudo in the past.

    You may try changing the group ownership of the /var/run/docker.sock file.

    sudo chown root:docker /var/run/docker.sock
    You may also try changing the group ownership of the ~/.docker directory.

    sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
    sudo chmod g+rwx "$HOME/.docker" -R
    And then try running docker with sudo. It should be fine.
```

Step3. ddocker load 出错 open /var/lib/docker/tmp/docker-import-837327978/bin/json: no such file or directory

```sh
    查阅资料发现这个由于 docker load 和 docker import 的区别导致.

    因为压缩包如果是用 docker save 打包的，就可以用 docker load，但是如果压缩包是用 docker export 打包的，那就需要用 docker import

    root@wohu:/$ docker load -i rabbitmq-1125.tar 
    open /var/lib/docker/tmp/docker-import-837327978/bin/json: no such file or directory
    root@wohu:/$ cat rabbitmq-1125.tar | docker import - rabbitmq_test:test
    sha256:23660698185e3f03fec12b82e50759f73385e5f16880d4b1dfd2c808282574e9
    ————————————————
    版权声明：本文为CSDN博主「wohu1104」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
    原文链接：https://blog.csdn.net/wohu1104/article/details/103334937
```



