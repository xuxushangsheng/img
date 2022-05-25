## Zookeeper 

zookeeper是一个开源的分布式协调服务，是由雅虎创建的，基于google chubby。

**zookeeper是分布式数据一致性的解决方案。**



#### zookeeper能做什么

数据的发布/订阅（配置中心）

负载均衡（dubbo利用zookeeper机制实现负载均衡）

命名服务

master选举

分布式队列

分布式锁



#### zookeeper的特性

* **顺序一致性**

  从同一个客户端发起的事务请求，最终会严格按照顺序被应用到zookeeper中

* **原子性**

  所有的事务请求的处理结果在整个集群中的所有机器上的应用情况是一致的，也就是说要么整个集群中的所有机器都成功应用了某一事务，要么全都不应用。

* **可靠性**

  一旦服务成功应用了某一个事务数据，并且对客户端做了响应，那么这个数据在整个集群中一定是同步并且保留下来的。

* **实时性（近实时）**

  一旦一个事务被成功应用，客户端就能够立即从服务器端读取到事务变更后的最新数据状态。



#### zookeeper的角色

* **leader**

  负责进行投票的发起和决议，更新系统状态

* **follower**

  用于接受客户端请求并想客户端返回结果，在选主过程中参与投票

* **observer**

  可以接受客户端连接，将写请求转发给leader，但observer不参加投票过程，只同步leader的状态，observer的目的是为了扩展系统，提高读取速度。

  ![img](https://images2015.cnblogs.com/blog/183233/201603/183233-20160316222444771-1363762533.png)

#### zookeeper安装

1. 下载zookeeper的安装包

2. 解压zookeeper

   ```sh
   tar -zxvf zookeeper-xxxx.tar.gz
   ```

3. cd 到 zk_home/conf，copy一份zoo.cfg

   ```shell
   cp zoo-sample.cfg zoo.cfg
   ```

4. 启动zookeeper

   ```shell
   sh zkServer.sh start
   ```

5. 启动客户端

   ```shell
   sh zkCli.sh [-server ip:port]
   ```

**集群环境**

zookeeper集群，包含三种角色：leader/follower/observer

observer 是一种特殊的zookeeper节点，可以帮助解决zookeeper的扩展性（如果大量客户端访问我们zookeeper集群，需要增加zookeeper集群机器数量，从而增加zookeeper集群的性能。导致zookeeper写性能下降，zookeeper的数据变更需要半数一闪的服务器投票通过。造成网络消耗增加投资成本）

1. 修改配置文件

   ```
   server.1 = ip1:2888:3181
   server.2 = ip2:2888:3181
   server.3 = ip3:2888:3181:observer
   ```

   id的取值范围：1-255；用id来标识该机器在集群中的机器序号

   2888：表示follower节点与leader节点交换信息的端口号

   3181：leader选举端口号

2. 创建myid

   在每一个服务器的dataDir目录下创建一个myid文件，文件就一行数据，数据内容是每台机器对应的server ID的数字

3. 启动zookeeper



#### zoo.cfg 参数

| 参数       | 说明                                                 |
| ---------- | ---------------------------------------------------- |
| tickTime   | zookeeper中最小的时间单位长度(ms)                    |
| initLimit  | follower节点启动后与leader节点完成数据同步的时间     |
| syncLimit  | leader节点和follower节点进行心跳检查的最大延迟时间   |
| dataDir    | zookeeper服务器存储快照文件的目录                    |
| dataLogDir | zookeeper事务日志的存储路径，默认指定在dataDir目录下 |
| clientPort | 客户端和服务器端建立连接的端口号                     |



#### zookeeper中的一些概念

1.数据模型

zookeeper的数据模型和文件系统类似，每一个节点称为：znode，是zookeeper中的最小数据单元。每一个znode上都可以保存数据和挂载子节点。从而构成一个层次化的属性结构。

2.节点类型

持久化节点：节点创建后会一直存在zookeeper服务器上，直到主动删除

持久化有序节点：每个节点都会为它的一级子节点维护一个顺序

临时节点：临时节点的生命周期和客户端的会话保持一致。当客户端会话失效，该节点自动淸理 

临时有序节点：在临时节点上多了一个顺序性特性

3.会话

。。。。。。。。



#### zookeeper常用命令

创建节点

```shell
create [-s] [-e] path data acl
-s:有序节点 -e:临时节点
```

**ACL**

zookeeper提供控制节点访问权限的功能，用于有效的保证zookeeper中数据的安全性。避免误操作而导致系统出现重大事故。

CREATE/READ/WRITE/DELETE/ADMIN



查看节点信息

``` shell
get path [watch]
```

**watcher**

zookeeper提供了分布式数据发布/订阅，zookeeper允许客户端向服务器注册一个watch监听。当服务器端的节点触发指定事件的时候会触发watcher。服务端会向客户端发送一个事件通知。



修改节点

```shell
set path data [version]
```

删除节点

```shell
delete path [version]
```



#### stat信息

| 参数           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| cversion       | 子节点的版本号                                               |
| aclVersion     | acl的版本号，修改节点权限                                    |
| dataVersion    | 当前节点数据的版本号                                         |
| czxid          | 节点被创建时的事务ID                                         |
| mzxid          | 节点最后一次被更新的事务ID                                   |
| pzxid          | 当前节点下的子节点最后一次被修改时的事务ID                   |
| ctime          | 创建时间                                                     |
| mtime          | 修改时间                                                     |
| ephemeralOwner | 创建临时节点的时候会有一个sessionID。该值存储<br />的就是这个sessionid |
| dataLength     | 数据值长度                                                   |
| numChildren    | 子节点数                                                     |

