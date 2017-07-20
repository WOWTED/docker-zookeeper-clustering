# docker-zookeeper-clustering
### 依赖
- Docker
- Docker Compose

### zookeeper 镜像下载
```bask
docker pull zookeeper
```
### 启动镜像
```bash
docker run --name my_zookeeper -d zookeeper:latest
```
- name 容器名称
- d 无页面运行
查看日志
```bash
docker logs -f my_zookeeper
```
### zkCli.sh 连接zookeeper
```bash
docker run -it --rm --link my_zookeeper:zookeeper zookeeper zkCli.sh -server zookeeper
```
### ZK 集群的搭建
- docker-compose 命令启动集群
docker-compose.yml
```
version: '2'
services:
    zoo1:
        image: zookeeper
        restart: always
        container_name: zoo1
        ports:
            - "2181:2181"
        environment:
            ZOO_MY_ID: 1
            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888

    zoo2:
        image: zookeeper
        restart: always
        container_name: zoo2
        ports:
            - "2182:2181"
        environment:
            ZOO_MY_ID: 2
            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888

    zoo3:
        image: zookeeper
        restart: always
        container_name: zoo3
        ports:
            - "2183:2181"
        environment:
            ZOO_MY_ID: 3
            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888
```
- 三个镜像对应端口  2181, 2182, 2183
- ZOO_MY_ID 表示 ZK 服务的 id ，范围为1-255 整数
- ZOO_SERVERS 集群列表

### 启动集群
```bash
COMPOSE_PROJECT_NAME=zk_test docker-compose up
```
### 查看
```bash
COMPOSE_PROJECT_NAME=zk_test docker-compose ps
```
>在 "docker-compose up" 和 "docker-compose ps" 前都添加了 COMPOSE_PROJECT_NAME=zk_test 这个环境变量, 这是为我们的 compose 工程起一个名字, 以免与其他的 compose 混淆.

### 使用 Docker 命令行客户端连接 ZK 集群
```bash
docker run -it --rm \
        --link zoo1:zk1 \
        --link zoo2:zk2 \
        --link zoo3:zk3 \
        --net zktest_default \
        zookeeper zkCli.sh -server zk1:2181,zk2:2181,zk3:2181
```
### 通过本地主机连接 ZK 集群
```bash
zkCli.sh -server localhost:2181,localhost:2182,localhost:2183
```
### 查看集群
我们可以通过 nc 命令连接到指定的 ZK 服务器, 然后发送 stat 可以查看 ZK 服务的状态, 例如:
```bash
echo stat | nc 127.0.0.1 2181
```
>Zookeeper version: 3.4.9-1757313, built on 08/23/2016 06:50 GMT
Clients:
 /172.18.0.1:49810[0](queued=0,recved=1,sent=0)
Latency min/avg/max: 5/39/74
Received: 4
Sent: 3
Connections: 1
Outstanding: 0
Zxid: 0x200000002
Mode: follower
Node count: 4

```bash
echo stat | nc 127.0.0.1 2182
```
>Zookeeper version: 3.4.9-1757313, built on 08/23/2016 06:50 GMT
Clients:
 /172.18.0.1:50870[0](queued=0,recved=1,sent=0)
Latency min/avg/max: 0/0/0
Received: 2
Sent: 1
Connections: 1
Outstanding: 0
Zxid: 0x200000002
Mode: follower
Node count: 4

```bash
echo stat | nc 127.0.0.1 2183
```
>Zookeeper version: 3.4.9-1757313, built on 08/23/2016 06:50 GMT
Clients:
 /172.18.0.1:51820[0](queued=0,recved=1,sent=0)
Latency min/avg/max: 0/0/0
Received: 2
Sent: 1
Connections: 1
Outstanding: 0
Zxid: 0x200000002
Mode: leader
Node count: 4
