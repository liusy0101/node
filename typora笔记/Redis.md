# 1、Redis集群模式



## （1）主从模式

**主从复制原理：**

- 从服务器连接主服务器，发送SYNC命令； 
- 主服务器接收到SYNC命名后，开始执行BGSAVE命令生成RDB文件并使用缓冲区记录此后执行的所有写命令； 
- 主服务器BGSAVE执行完后，向所有从服务器发送快照文件，并在发送期间继续记录被执行的写命令； 
- 从服务器收到快照文件后丢弃所有旧数据，载入收到的快照； 
- 主服务器快照发送完毕后开始向从服务器发送缓冲区中的写命令； 
- 从服务器完成对快照的载入，开始接收命令请求，并执行来自主服务器缓冲区的写命令；（**从服务器初始化完成**）
- 主服务器每执行一个写命令就会向从服务器发送相同的写命令，从服务器接收并执行收到的写命令（**从服务器初始化完成后的操作**）

 ![img](typora-user-images/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NlbGVjdGdvb2Rib3k=,size_16,color_FFFFFF,t_70) 



**主从复制优缺点：**

**优点：**

- 支持主从复制，主机会自动将数据同步到从机，可以进行读写分离
- 为了分载Master的读操作压力，Slave服务器可以为客户端提供只读操作的服务，写服务仍然必须由Master来完成
- Slave同样可以接受其它Slaves的连接和同步请求，这样可以有效的分载Master的同步压力。
- Master Server是以非阻塞的方式为Slaves提供服务。所以在Master-Slave同步期间，客户端仍然可以提交查询或修改请求。
- Slave Server同样是以非阻塞的方式完成数据同步。在同步期间，如果有客户端提交查询请求，Redis则返回同步之前的数据

**缺点：**

- Redis不具备自动容错和恢复功能，主机从机的宕机都会导致前端部分读写请求失败，需要等待机器重启或者手动切换前端的IP才能恢复。
- 主机宕机，宕机前有部分数据未能及时同步到从机，切换IP后还会引入数据不一致的问题，降低了系统的可用性。
- Redis较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂。



## （2）哨兵模式

 （哨兵）是Redis的高可用性解决方案：由一个或多个Sentinel实例组成的Sentinel系统可以监视任意多个主服务器，以及这些主服务器属下的所有从服务器，并在被监视的主服务器进入下线状态时，自动将下线主服务器属下的某个从服务器升级为新的主服务器。 

 ![img](typora-user-images/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV10L3NlbGVjdGdvb2Rib3k=,size_16,color_FFFFFF,t_70) 

**哨兵的工作方式：**

- 每个Sentinel（哨兵）进程以每秒钟一次的频率向整个集群中的Master主服务器，Slave从服务器以及其他Sentinel（哨兵）进程发送一个 PING 命令。
- 如果一个实例（instance）距离最后一次有效回复 PING 命令的时间超过 down-after-milliseconds 选项所指定的值， 则这个实例会被 Sentinel（哨兵）进程标记为主观下线（SDOWN）
- 如果一个Master主服务器被标记为主观下线（SDOWN），则正在监视这个Master主服务器的所有 Sentinel（哨兵）进程要以每秒一次的频率确认Master主服务器的确进入了主观下线状态
- 当有足够数量的 Sentinel（哨兵）进程（大于等于配置文件指定的值）在指定的时间范围内确认Master主服务器进入了主观下线状态（SDOWN）， 则Master主服务器会被标记为客观下线（ODOWN）
- 在一般情况下， 每个 Sentinel（哨兵）进程会以每 10 秒一次的频率向集群中的所有Master主服务器、Slave从服务器发送 INFO 命令。
- 当Master主服务器被 Sentinel（哨兵）进程标记为客观下线（ODOWN）时，Sentinel（哨兵）进程向下线的 Master主服务器的所有 Slave从服务器发送 INFO 命令的频率会从 10 秒一次改为每秒一次。
- 若没有足够数量的 Sentinel（哨兵）进程同意 Master主服务器下线， Master主服务器的客观下线状态就会被移除。若 Master主服务器重新向 Sentinel（哨兵）进程发送 PING 命令返回有效回复，Master主服务器的主观下线状态就会被移除。



**哨兵模式的优缺点**

**优点：**

- 哨兵模式是基于主从模式的，所有主从的优点，哨兵模式都具有。
- 主从可以自动切换，系统更健壮，可用性更高。

**缺点：**

- Redis较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂。



选举sentinel领导者 使用的raft算法(https://raft.github.io)，大致思路：

1. 每个做主观下线的sentinel节点像其他sentinel节点发送命令，要求将自己设置为领导者
2. 接收到的sentinel可以同意或者拒绝

3. 如果该sentinel节点发现自己的票数已经超过半数并且超过了quorum
4. 如果此过程选举出了多个领导者，那么将等待一段时重新进行选举



主节点选举：选举出可以代替主节点的slave从节点

- 1、选择健康状态从节点（排除主观下线、断线），排除5秒钟没有心跳的、排除主节点失联超过10*down-after-millisecends
- 2、选择slave-priority高的从节点优先级列表
- 4、选择偏移量大的
- 5、选择runid小的



进行故障转移

- 1、sentinel的领导者从slave中选举出合适的从节点进行故障转移
- 2、对选取的slave执行slave of no one
- 3、更新应用程序端的链接到新的主节点

- 4、对其他从节点变更master为新的节点

- 5、修复原来的master并将其设置为新master的slave



## （3）Cluster模式

Redis-Cluster采用无中心结构,它的特点如下：

- 所有的redis节点彼此互联(PING-PONG机制),内部使用二进制协议优化传输速度和带宽。
- 节点的fail是通过集群中超过半数的节点检测失效时才生效。
- 客户端与redis节点直连,不需要中间代理层.客户端不需要连接集群所有节点,连接集群中任何一个可用节点即可。

**工作方式：**

在redis的每一个节点上，都有这么两个东西，一个是插槽（slot），它的的取值范围是：0-16383。还有一个就是cluster，可以理解为是一个集群管理的插件。当我们的存取的key到达的时候，redis会根据crc16的算法得出一个结果，然后把结果对 16384 求余数，这样每个 key 都会对应一个编号在 0-16383 之间的哈希槽，通过这个值，去找到对应的插槽所对应的节点，然后直接自动跳转到这个对应的节点上进行存取操作。

为了保证高可用，redis-cluster集群引入了主从模式，一个主节点对应一个或者多个从节点，当主节点宕机的时候，就会启用从节点。当其它主节点ping一个主节点A时，如果半数以上的主节点与A通信超时，那么认为主节点A宕机了。如果主节点A和它的从节点A1都宕机了，那么该集群就无法再提供服务了。





**数据迁移方式**

Redis使用了哈希槽的概念，并没有使用一致性hash。

数据迁移是slot（槽）和key的迁移，此功能可实现平滑的扩容和缩容。

 现在要将Master A节点中编号为1、2、3的slot迁移到Master B节点中，在slot迁移的中间状态下，slot 1、2、3在Master A节点的状态表现为MIGRATING（迁移）,在Master B节点的状态表现为IMPORTING（入口）。 

 ![img](typora-user-images/20180103174740149) 

**此时并不会刷新node的映射关系**



**IMPORTING状态**

被迁移slot在目标Master B节点中出现的一种状态 ，准备迁移slot从Mater A到Master B的时候，被迁移slot的状态首先变为IMPORTING状态。 



**键空间迁移**

 键空间迁移是指当满足了slot迁移前提的情况下，通过相关命令将slot 1、2、3中的键空间从Master A节点转移到Master B节点。此时刷新node的映射关系。 

 ![img](typora-user-images/20180103174824673) 



# 2、Redis持久化方式

Redis提供了两种持久化方式，分别是RDB和AOF。



**一、RDB（保存数据库键对值）**

（1）redis默认开启了RDB持久化方式

```
#下面这一行取消注释，下面三行注释掉，就是关闭RDB
#save ""
#下面三行是开启RDB持久化方式
save 900 1
save 300 10
save 60 10000
```

开启之后，会生成一个dump.rdb文件，这个就是数据落在磁盘上的实际存储文件。下面是对这三行配置的解读。

```
#每900秒内，数据库至少有1次修改就进行保存 
save 900 1
#每300 秒内，数据库至少有10次修改就进行保存
save 300 10
#每60 秒内，数据库至少有10000次修改就进行保存
save 60 10000
```

（2）开启RDB持久化之后，数据库启动的时候就会先加载dump.rdb文件进内存 ，这个过程是阻塞的

![图片](https://mmbiz.qpic.cn/mmbiz_png/1WZDcbq4okjDfwMaBavMQicokjSc4pu41WkzFgMeibb1riaJXUZGA6MAuykFjEjyqK06CyImT43pdKu8UUVQdnxuA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

（3）执行***SAVE***或***BGSAVE***命令可以触发RDB方式数据落盘保存

SAVE命令是阻塞的，BGSAVE命令是非阻塞的，其实上述的几秒内有至少几次修改就保存的后台也是执行的BGSAVE命令。

![图片](https://mmbiz.qpic.cn/mmbiz_png/1WZDcbq4okjDfwMaBavMQicokjSc4pu41sqLOL9F8gibJ6MpzqX9e8XxjLLX478N6vRpuyWicLiaDBstUOZt2CSwIg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





**二、AOF（Append Only File，保存数据库执行过的命令）**

（1）开启AOF持久化非常简单，配置一下两个配置项就行

```
#开启aof命令
appendonly yes
#aof文件名称
appendfilename "appendonly.aof"
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/1WZDcbq4okjDfwMaBavMQicokjSc4pu41viadNwU47ozVa1amUcW6iazxyFmzXgMUia8uQ6tuhCwNyaPZPDdPCaY7w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

（2）开启AOF之后，redis启动会去加载aof文件，但是不会去加载rdb文件，因为aof文件的更新速度比rdb文件快，所以redis会优先加载aof文件，只有当aof没开启时，才会去加载rdb文件。

![图片](https://mmbiz.qpic.cn/mmbiz_png/1WZDcbq4okjDfwMaBavMQicokjSc4pu41AibcpPQOKkbySv9TXq0cg2UBVribMZopB3rW1hIYAiclAIEiaMWibibdNqNA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/1WZDcbq4okjDfwMaBavMQicokjSc4pu414wv7GzZWTfjHp0qSupzwOnqiaUbic5ic0icR1he2bCZ444aV01sZmJpSQw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





































