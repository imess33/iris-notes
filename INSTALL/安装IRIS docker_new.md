修改几个地方

- 安装community版本不用authentication.

- volume映射的内部从/dur变成/external

- 两个网页

  https://docs.intersystems.com/components/csp/docbook/Doc.View.cls?KEY=PAGE_containerregistry#PAGE_containerregistry

  

[Deploy InterSystems IRIS Community Edition on Your Own System](https://docs.intersystems.com/irislatest/csp/docbook/Doc.View.cls?KEY=ACLOUD#ACLOUD_image)

### 记录

```sh

CNMBPHMA:simpleBule hma$ docker pull containers.intersystems.com/intersystems/irishealth-ml-community:2022.1.0.209.0
2022.1.0.209.0: Pulling from intersystems/irishealth-ml-community
...
CNMBPHMA:simpleBule hma$ docker run --name iris -d -p 1972:1972 -p 52773:52773 -v /Users/hma/InterSystems/external/iriscommunity:/external containers.intersystems.com/intersystems/irishealth-ml-community:2022.1.0.209.0
af34ddc4655bfbec268ca6f135c6f7af2f15ff1b2fd63e0b9e06efe28f20bd6a
CNMBPHMA:simpleBule hma$ docker ps
CONTAINER ID   IMAGE                                                                             COMMAND                  CREATED         STATUS                            PORTS                                                                              NAMES
af34ddc4655b   containers.intersystems.com/intersystems/irishealth-ml-community:2022.1.0.209.0   "/tini -- /iris-main"    5 seconds ago   Up 4 seconds (health: starting)   0.0.0.0:1972->1972/tcp, 2188/tcp, 53773/tcp, 0.0.0.0:52773->52773/tcp, 54773/tcp   iris
9d0e67cc0118   mysql:latest                                                                      "docker-entrypoint.s…"   2 hours ago     Up 2 hours                        0.0.0.0:3306->3306/tcp, 33060/tcp                                                  mysql
CNMBPHMA:simpleBule hma$
```

新iris第一次登录要设置密码：demo1234
