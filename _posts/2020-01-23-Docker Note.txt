---
date: 2020-01-23 16:28
status: public
title: '2020-01-23-Docker Note'
---

# top use
docker build -t="temp/name" .
docker run -t -i temp/name /bin/bash
docker run -itd --name redis-test -p 6379:6379 redis
docker attach 容器ID  
docker exec -it 容器ID /bin/bash

# 时区
方式一：
environment:
  - SET_CONTAINER_TIMEZONE=true
  - CONTAINER_TIMEZONE=Asia/Shanghai
	  
方式二：
environment:
  - TZ=Asia/Shanghai

# debug的坑
从Java 9开始,JDWP套接字连接器默认只接受本地连接.
要加上*:5005 (注意真机不要这样)
[一些建议](https://www.oschina.net/translate/why-you-dont-need-to-run-sshd-in-docker?cmp )
windows的volume，/c/开始表示c盘]
# Docker API启动
编辑vim /lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375
systemctl --system daemon-reload
systemctl enable docker
systemctl start docker
service docker restart
docker -H 127.0.0.1:2375 info
netstat -tunlp
# mirror
if you want to pull 
docker pull tomcat:latest
docker pull hub.c.163.com/library/tomcat:latest
# 安装
https://www.runoob.com/docker/ubuntu-docker-install.html

# Docker证书
```bash
which openssl
mkdir /etc/docker
echo -1|sudo tee ca.srl
sudo opensll genrsa -des3 -out ca-key.pem
sudo openssl req -new -x509 -days 365 -key ca-key.pem -out ca.pem
sudo opensll genrsa -des3 -out server-key.pem
sudo opensll req -new -key server-key.pem -out server.csr
# 注意CommonName时要输入* 或者服务器域名
sudo openssl x509 -req -days 365 -in server.csr -CA ca.pem -CAkey ca-key.pem -out server-cert.pem
sudo opensll rsa -in server-key.pem -out server-key.pem
sudo chmod 0600 /etc/docker.server-key.pem /etc.docker/server-cert.pem /etc/docker/ca-key.pem /etc/docker/ca.pem
# 设置Docker配置文件/etc/default/docker
ExecStart=/usr/bin/docker -d -H tcp://0.0.0.0:2379 --tlsverify
--tlscacert=/etc/docker/ca.pem --tlscert=/etc/docket/setvet-cert.pem --tlskey=/etc/docker/server-key.pem
sudo systemctl --system daemon-reload
# 客户端证书
sudo openssl genrsa -des3 -out client-key. pem
sudo openssl req -new -key client-key. pem -out client.csr
echo extendedKeyUsage = clientAuth > extfile. cnf
sudo openssl x509 -req -days 365 -in client.csr -CA ca.pem  -CAkey ca- key. pem -out client- cert.pem -extfile extfile.cnf
sudo openssl rsa -in client- key. pem -out client- key. pem
# 拷贝到.docker文件夹下，并执行
chmod 0600 ～/. docker/ key. pem ～/. docker/ cert. pem
# 最后采用 --tlsverify来连接
sudo docker -H= docker. example. com: 2376 --tlsverify info
```
# docker-compose 

## yml例子
```yml
#create a network and add our services into this network:
#so, "app" service will be able to connect to the mysql database from "db" servoce by the hostname="db":
#jdbc:mysql://db:3306/DOCKERDB

#Connection url for connection in the DatabaseView:
#  jdbc:mysql://0.0.0.0:13306/DOCKERDB, login=root, password=root
#App is available at: http://localhost:18080/entitybus/post
version: "2.1"

networks:
  test:

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "18080:8080"
    depends_on:
      db:
        condition: service_healthy
    networks:
      - test

  db:
    image: opavlova/db-mysql:5.7-test
    container_name: db
    ports:
      - "13306:3306"
    healthcheck:
          test: ["CMD", "mysql", "-h", "localhost", "-P", "3306", "-u", "root", "--password=root", "-e", "select 1", "DOCKERDB"]
          interval: 1s
          timeout: 3s
          retries: 30
    networks:
      - test
```
## 参考
[jetBrain使用参考](https://www.jetbrains.com/help/idea/run-and-debug-a-spring-boot-application-using-docker-compose.html )
[jetBrainDocker配置](https://www.jetbrains.com/help/idea/docker.html# )
[官网的参考文档](https://docs.docker.com/compose/compose-file/ )
[较为准确的中文翻译版](https://deepzz.com/post/docker-compose-file.html#toc_31 )
[Dockerfile参考文档](https://deepzz.com/post/dockerfile-reference.html )
[使用查询](https://www.cnblogs.com/ray-mmss/p/10868754.html )
```bash
#查看帮助
docker-compose -h
# -f  指定使用的 Compose 模板文件，默认为 docker-compose.yml，可以多次指定。
docker-compose -f docker-compose.yml up -d 
#启动所有容器，-d 将会在后台启动并运行所有的容器
docker-compose up -d
#停用移除所有容器以及网络相关
docker-compose down
#查看服务容器的输出
docker-compose logs
#列出项目中目前的所有容器
docker-compose ps
#构建（重新构建）项目中的服务容器。服务容器一旦构建后，将会带上一个标记名，例如对于 web 项目中的一个 db 容器，可能是 web_db。可以随时在项目目录下运行 docker-compose build 来重新构建服务
docker-compose build
#拉取服务依赖的镜像
docker-compose pull
#重启项目中的服务
docker-compose restart
#删除所有（停止状态的）服务容器。推荐先执行 docker-compose stop 命令来停止容器。
docker-compose rm 
#在指定服务上执行一个命令。
docker-compose run ubuntu ping docker.com
#设置指定服务运行的容器个数。通过 service=num 的参数来设置数量
docker-compose scale web=3 db=2
#启动已经存在的服务容器。
docker-compose start
#停止已经处于运行状态的容器，但不删除它。通过 docker-compose start 可以再次启动这些容器。
docker-compose stop
```