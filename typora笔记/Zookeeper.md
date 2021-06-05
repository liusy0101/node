[toc]



# Zookeeper

Zookeeper是一个分布式协调服务，可用于服务发现，分布式锁，分布式leader选举，配置管理等。



## 1、Zookeeper角色

Zookeeper集群是一个基于主从复制的高可用集群。

 ![img](typora-user-images/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgwMzMyOQ==,size_16,color_FFFFFF,t_70) 

### （1）leader（领导者）

为客户端提供读和写的功能，负责投票的发起和决议，集群里面只有leader才能接受写的服务。

### （2）follower（跟随者）

为客户端提供读服务，如果是写的服务则转发给leader。在选举过程中进行投票。

### （3）observer（观察者）

为客户端提供读服务，如果是写服务就转发个leader。不参与leader的选举投票。也不参与写的过半原则机制。在不影响写的前提下，提高集群读的性能，此角色于zookeeper3.3系列新增的角色。

这个角色设置是在zoo.cfg文件中新增配置：

peerType=observer

server.n=host:2181:3181:observer //n是集群中的机器序号



## 2、Zookeeper节点

zookeeper的数据模型和文件系统类似，每一个节点称为：znode，是zookeeper中的最小数据单元。每一个znode上都可以保存数据和挂载子节点。从而构成一个层次化的属性结构

### （1）节点特性

**持久化节点  ：** 节点创建后会一直存在zookeeper服务器上，直到主动删除

**持久化有序节点 ：**每个节点都会为它的一级子节点维护一个顺序

**临时节点 ：** 临时节点的生命周期和客户端的会话保持一致。当客户端会话失效，该节点自动清理，临时节点下不可创建子节点。

**临时有序节点 ：** 在临时节点上多了一个顺序性特性 



## 3、事务编号Zxid 和Epoch

 （事务请求计数器 + epoch）

### （1）Zxid

在ZAB协议的事务编号Zxid设计中

Zxid是一个64位的数字，其中低32位是一个简单的单调递增的计数器，针对客户端每个事务请求，计数器加1

高32位代表Leader周期epoch的编号，每个当选产生一个新的leader服务器，就会从这个 Leader 服务器上取出其本地日志中最大事务的ZXID，并从中读取 epoch 值，然后加 1，以此作为新的 epoch，并将低 32 位从 0 开始计数。 

Zxid（Transaction id）类似于RDBMS中的事务ID，用于标识一次更新操作的Proposal（提议）ID

为保证顺序性，该Zxid必须单调递增。



### （2）epoch

可以理解为当前集群所处的年代或者周期，每个 leader 就像皇帝，都有自己的年号，所以每次改朝换代，leader 变更之后，都会在前一个年代的基础上加 1。这样就算旧的 leader 崩溃 恢复之后，也没有人听他的了，因为 follower 只听从当前年代的 leader 的命令。



## 4、ZAB协议

Zookeeper Atomic Broadcast：Zookeeper原子广播

Zookeeper通过Zab协议来保证分布式事务的最终一致性

- ZAB协议是为了分布式协调服务Zookeeper专门设计的一种支持崩溃恢复的原子广播协议，是ZK保证数据一致性的核心算法。借鉴了Paxos算法，但不像Paxos是一种通用的分布式的一致性算法
- 在ZK中主要依赖ZAB协议来实现数据一致性，基于该协议，ZK实现了一种主备模型（即Leader和Follower）的系统架构来保证集群中每个副本之间数据的一致性。
  - 主备模型就是Leader负责处理外部的写事务请求，同事将数据同步到其他Follower节点。



 Zookeeper 客户端会随机的链接到 zookeeper 集群中的一个节点，如果是读请求，就直接从当前节点中读取数据；如果是写请求，那么节点就会向 Leader 提交事务，Leader 接收到事务提交，会广播该事务，只要超过半数节点写入成功，该事务就会被提交。 



### （1）ZAB协议特性

1）确保那些已经在 Leader 服务器上提交（Commit）的事务最终被所有的服务器提交。
2）确保丢弃那些只在 Leader 上被提出而没有被提交的事务。

 ![img](typora-user-images/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80M12DgwMzMyOQ==,size_16,color_FFFFFF,t_70) 



### （2）实现的作用

1）使用一个单一的主进程（Leader）来接收并处理客户端的事务请求（也就是写请求），并采用了Zab的原子广播协议，将服务器数据的状态变更以 事务proposal （事务提议）的形式广播到所有的副本（Follower）进程上去。

2）保证一个全局的变更序列被顺序引用。
Zookeeper是一个树形结构，很多操作都要先检查才能确定是否可以执行，比如P1的事务t1可能是创建节点"/a"，t2可能是创建节点"/a/bb"，只有先创建了父节点"/a"，才能创建子节点"/a/b"。

为了保证这一点，Zab要保证同一个Leader发起的事务要按顺序被apply，同时还要保证只有先前Leader的事务被apply之后，新选举出来的Leader才能再次发起事务。

当主进程出现异常的时候，整个zk集群依旧能正常工作。



### （3）协议原理

ZAB协议要求每个Leader都要经历三个阶段：发现、同步、广播

**发现：**

要求ZK集群必须选举出来一个Leader进程，同时Leader会维护一个Follower可用客户端列表，将来客户端可以和这些Follower节点进行通信

**同步：**

 Leader 要负责将本身的数据与 Follower 完成同步，做到多副本存储。这样也是提现了CAP中的高可用和分区容错。Follower将队列中未处理完的请求消费完成后，写入本地事务日志中。 

**广播：**

 Leader 可以接受客户端新的事务Proposal请求，将新的Proposal请求广播给所有的 Follower。 



**ZAB节点有三种状态：**

Following：当前节点是跟随者，服从 Leader 节点的命令。

Leading：当前节点是 Leader，负责协调事务。

Election/Looking：节点处于选举状态，正在寻找 Leader。

代码实现中，多了一种状态：Observing 状态
这是 Zookeeper 引入 Observer 之后加入的，Observer 不参与选举，是只读节点，跟 Zab 协议没有关系。



**节点的持久状态：**

history：当前节点接收到事务 Proposal 的Log

acceptedEpoch：Follower 已经接受的 Leader 更改 epoch 的 newEpoch 提议。

currentEpoch：当前所处的 Leader 年代

lastZxid：history 中最近接收到的Proposal 的 zxid（最大zxid）



### （4）协议核心

核心：**定义了事务请求的处理方式**

1）所有的事务请求必须由一个全局唯一的服务器来协调处理，这样的服务器被叫做 Leader服务器。其他剩余的服务器则是 Follower服务器。

2）Leader服务器 负责将一个客户端事务请求，转换成一个 事务Proposal，并将该 Proposal 分发给集群中所有的 Follower 服务器，也就是向所有 Follower 节点发送数据广播请求（或数据复制）

3）分发之后Leader服务器需要等待所有Follower服务器的反馈（Ack请求），在Zab协议中，只要超过半数的Follower服务器进行了正确的反馈后（也就是收到半数以上的Follower的Ack请求），那么 Leader 就会再次向所有的 Follower服务器发送 Commit 消息，要求其将上一个 事务proposal 进行提交。

 ![img](typora-user-images/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR10cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgwMzMyOQ==,size_16,color_FFFFFF,t_70) 



### （5）ZAB协议内容

ZAB协议包括两种基本的模式： **崩溃恢复** 和 **消息广播**

#### 1、协议过程

当整个集群启动过程中，或者当 Leader 服务器出现网络中弄断、崩溃退出或重启等异常时，Zab协议就会 进入崩溃恢复模式，选举产生新的Leader。

当选举产生了新的 Leader，同时集群中有过半的机器与该 Leader 服务器完成了状态同步（即数据同步）之后，Zab协议就会退出崩溃恢复模式，进入消息广播模式。

这时，如果有一台遵守Zab协议的服务器加入集群，因为此时集群中已经存在一个Leader服务器在广播消息，那么该新加入的服务器自动进入恢复模式：找到Leader服务器，并且完成数据同步。同步完成后，作为新的Follower一起参与到消息广播流程中。



#### 2、协议状态切换

 当Leader出现崩溃退出或者机器重启，亦或是集群中不存在超过半数的服务器与Leader保存正常通信，Zab就会再一次进入崩溃恢复，发起新一轮Leader选举并实现数据同步。同步完成后又会进入消息广播模式，接收事务请求。 



#### 3、保证消息有序

在整个消息广播中，Leader会将每一个事务请求转换成对应的 proposal 来进行广播，并且在广播事务Proposal 之前，Leader服务器会首先为这个事务Proposal分配一个全局单递增的唯一ID，称之为事务ID（即zxid），由于Zab协议需要保证每一个消息的严格的顺序关系，因此必须将每一个proposal按照其zxid的先后顺序进行排序和处理。



#### 4、消息广播

1）在zookeeper集群中，数据副本的传递策略就是采用消息广播模式。zookeeper中数据副本的同步方式与二段提交相似，但是却又不同。二段提交要求协调者必须等到所有的参与者全部反馈ACK确认消息后，再发送commit消息。要求所有的参与者要么全部成功，要么全部失败。二段提交会产生严重的阻塞问题。

2）Zab协议中 Leader 等待 Follower 的ACK反馈消息是指“只要半数以上的Follower成功反馈即可，不需要收到全部Follower反馈”

 ![img](typora-user-images/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG·4ubmV0L3dlaXhpbl80MDgwMzMyOQ==,size_16,color_FFFFFF,t_70) 



消息广播具体步骤

1）客户端发起一个写操作请求。

2）Leader 服务器将客户端的请求转化为事务 Proposal 提案，同时为每个 Proposal 分配一个全局的ID，即zxid。

3）Leader 服务器为每个 Follower 服务器分配一个单独的队列，然后将需要广播的 Proposal 依次放到队列中取，并且根据 FIFO 策略进行消息发送。

4）Follower 接收到 Proposal 后，会首先将其以事务日志的方式写入本地磁盘中，写入成功后向 Leader 反馈一个 Ack 响应消息。

5）Leader 接收到超过半数以上 Follower 的 Ack 响应消息后，即认为消息发送成功，可以发送 commit 消息。

6）Leader 向所有 Follower 广播 commit 消息，同时自身也会完成事务提交。Follower 接收到 commit 消息后，会将上一条事务提交。



zookeeper 采用 Zab 协议的核心，就是只要有一台服务器提交了 Proposal，就要确保所有的服务器最终都能正确提交 Proposal。这也是 CAP/BASE 实现最终一致性的一个体现。

Leader 服务器与每一个 Follower 服务器之间都维护了一个单独的 FIFO 消息队列进行收发消息，使用队列消息可以做到异步解耦。 Leader 和 Follower 之间只需要往队列中发消息即可。如果使用同步的方式会引起阻塞，性能要下降很多。





#### 5、崩溃恢复

一旦 Leader 服务器出现崩溃或者由于网络原因导致 Leader 服务器失去了与过半 Follower 的联系，那么就会进入崩溃恢复模式。

在 Zab 协议中，为了保证程序的正确运行，整个恢复过程结束后需要选举出一个新的 Leader 服务器。因此 Zab 协议需要一个高效且可靠的 Leader 选举算法，从而确保能够快速选举出新的 Leader 。

Leader 选举算法不仅仅需要让 Leader 自己知道自己已经被选举为 Leader ，同时还需要让集群中的所有其他机器也能够快速感知到选举产生的新 Leader 服务器。

崩溃恢复主要包括两部分：Leader选举 和 数据恢复





### （6）保证数据一致性

假设两种异常情况：
1、一个事务在 Leader 上提交了，并且过半的 Folower 都响应 Ack 了，但是 Leader 在 Commit 消息发出之前挂了。
2、假设一个事务在 Leader 提出之后，Leader 挂了。

要确保如果发生上述两种情况，数据还能保持一致性，那么 Zab 协议选举算法必须满足以下要求：

Zab 协议崩溃恢复要求满足以下两个要求：
1）确保已经被 Leader 提交的 Proposal 必须最终被所有的 Follower 服务器提交。
2）确保丢弃已经被 Leader 提出的但是没有被提交的 Proposal。

根据上述要求
Zab协议需要保证选举出来的Leader需要满足以下条件：
1）新选举出来的 Leader 不能包含未提交的 Proposal 。
即新选举的 Leader 必须都是已经提交了 Proposal 的 Follower 服务器节点。
2）新选举的 Leader 节点中含有最大的 zxid 。
这样做的好处是可以避免 Leader 服务器检查 Proposal 的提交和丢弃工作。



#### 1、如何进行数据同步

1）完成 Leader 选举后（新的 Leader 具有最高的zxid），在正式开始工作之前（接收事务请求，然后提出新的 Proposal），Leader 服务器会首先确认事务日志中的所有的 Proposal 是否已经被集群中过半的服务器 Commit。

2）Leader 服务器需要确保所有的 Follower 服务器能够接收到每一条事务的 Proposal ，并且能将所有已经提交的事务 Proposal 应用到内存数据中。等到 Follower 将所有尚未同步的事务 Proposal 都从 Leader 服务器上同步过啦并且应用到内存数据中以后，Leader 才会把该 Follower 加入到真正可用的 Follower 列表中。



**Zab 数据同步过程中，如何处理需要丢弃的 Proposal**



在 Zab 的事务编号 zxid 设计中，zxid是一个64位的数字。

其中低32位可以看成一个简单的单增计数器，针对客户端每一个事务请求，Leader 在产生新的 Proposal 事务时，都会对该计数器加1。而高32位则代表了 Leader 周期的 epoch 编号。

epoch 编号可以理解为当前集群所处的年代，或者周期。每次Leader变更之后都会在 epoch 的基础上加1，这样旧的 Leader 崩溃恢复之后，其他Follower 也不会听它的了，因为 Follower 只服从epoch最高的 Leader 命令。

每当选举产生一个新的 Leader ，就会从这个 Leader 服务器上取出本地事务日志充最大编号 Proposal 的 zxid，并从 zxid 中解析得到对应的 epoch 编号，然后再对其加1，之后该编号就作为新的 epoch 值，并将低32位数字归零，由0开始重新生成zxid。

Zab 协议通过 epoch 编号来区分 Leader 变化周期，能够有效避免不同的 Leader 错误的使用了相同的 zxid 编号提出了不一样的 Proposal 的异常情况。

当一个包含了上一个 Leader 周期中尚未提交过的事务 Proposal 的服务器启动时，当这台机器加入集群中，以 Follower 角色连上 Leader 服务器后，Leader 服务器会根据自己服务器上最后提交的 Proposal 来和 Follower 服务器的 Proposal 进行比对，比对的结果肯定是 Leader 要求 Follower 进行一个回退操作，回退到一个确实已经被集群中过半机器 Commit 的最新 Proposal。



### （7）4个阶段

#### 1、Leader election 选举阶段

 节点在一开始都处于选举阶段，只要有一个节点得到超半数 节点的票数，它就可以当选准 leader。只有到达广播阶段（broadcast） 准 leader才会成为真正的leader。这一阶段的目的是就是为了选出一个准 leader，然后进入下一个阶段。 

 **Zookeeper 规定所有有效的投票都必须在同一个 轮次 中，每个服务器在开始新一轮投票时，都会对自己维护的 logicalClock 进行自增操作**。 

每个服务器在广播自己的选票前，会将自己的投票箱（recvset）清空。该投票箱记录了所受到的选票。
例如：Server_2 投票给 Server_3，Server_3 投票给 Server_1，则Server_1的投票箱为(2,3)、(3,1)、(1,1)。（每个服务器都会默认给自己投票）

前一个数字表示投票者，后一个数字表示被选举者。票箱中只会记录每一个投票者的最后一次投票记录，如果投票者更新自己的选票，则其他服务器收到该新选票后会在自己的票箱中更新该服务器的选票。

 ![img](typora-user-images/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgw1MzMyOQ==,size_16,color_FFFFFF,t_70) 





#### 2、Discover 发现阶段 

  接受提议、生成 epoch 、接受 epoch 

在这个阶段，followers 跟准 leader 进行通信，同步 followers 最近接收的事务提议。这个一阶段的主要目的是发现当前大多数节点接收的最新提议，并且 准 leader 生成新的 epoch，让 followers 接受，更新它们的 accepted Epoch 

一个 follower 只会连接一个 leader，如果有一个节点 f 认为另一个 follower p 是 leader，f 在尝试连接 p 时会被拒绝，f 被拒绝之后，就会进入重新选举阶段。

 <img src="typora-user-images/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV01L3dlaXhpbl80MDgwMzMyOQ==,size_16,color_FFFFFF,t_70" alt="img"  /> 



#### 3、Synchronization 同步阶段

 同步 follower副本  

 同步阶段主要是利用 leader 前一阶段获得的最新提议历史， 同步集群中所有的副本。只有当大多数节点都同步完成，准 leader 才会成为真正的 leader。 follower 只会接收 zxid 比自己的 lastZxid 大的提议。 

 ![img](typora-user-images/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgwMzMyOQ==,size_16,co2lor_FFFFFF,t_70) 



#### 4、Broadcast 广播阶段

到了这个阶段，Zookeeper 集群才能正式对外提供事务服务， 并且 leader 可以进行消息广播。同时如果有新的节点加入，还需要对新节点进行同步。

ZAB 提交事务并不像 2PC 一样需要全部 follower 都 ACK，只需要得到超过半数的节点的 ACK 就 可以了。

 ![img](typora-user-images/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR01cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgwMzMyOQ==,size_16,color_FFFFFF,t_70) 



### （8）ZAB协议JAVA

协议的 Java 版本实现跟上面的定义略有不同，选举阶段使用的是 Fast Leader Election（FLE），它包含了步骤1的发现指责。因为FLE会选举拥有最新提议的历史节点作为 Leader，这样就省去了发现最新提议的步骤。

实际的实现将发现和同步阶段合并为 Recovery Phase（恢复阶段），所以，Zab 的实现实际上有三个阶段。

Zab协议三个阶段：

1）选举（Fast Leader Election）
2）恢复（Recovery Phase）
3）广播（Broadcast Phase）

Fast Leader Election（快速选举）
前面提到的 FLE 会选举拥有最新Proposal history （lastZxid最大）的节点作为 Leader，这样就省去了发现最新提议的步骤。这是基于拥有最新提议的节点也拥有最新的提交记录

成为 Leader 的条件：
1）选 epoch 最大的
2）若 epoch 相等，选 zxid 最大的
3）若 epoch 和 zxid 相等，选择 server_id 最大的（zoo.cfg中的myid）

节点在选举开始时，都默认投票给自己，当接收其他节点的选票时，会根据上面的 Leader条件 判断并且更改自己的选票，然后重新发送选票给其他节点。当有一个节点的得票超过半数，该节点会设置自己的状态为 Leading ，其他节点会设置自己的状态为 Following。

 ![img](typora-user-images/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgwM1zMyOQ==,size_16,color_FFFFFF,t_70) 



Recovery Phase（恢复阶段）
这一阶段 Follower 发送他们的 lastZxid 给 Leader，Leader 根据 lastZxid 决定如何同步数据。这里的实现跟前面的 Phase 2 有所不同：Follower 收到 TRUNC 指令会终止 L.lastCommitedZxid 之后的 Proposal ，收到 DIFF 指令会接收新的 Proposal。

history.lastCommitedZxid：最近被提交的 Proposal zxid
history.oldThreshold：被认为已经太旧的已经提交的 Proposal zxid

 ![img](typora-user-images/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgwMzMyOQ==,size_116,color_FFFFFF,t_70) 





 **事务操作**

ZAB协议对于事务操作的处理是一个类似于二阶段提交过程。针对客户端的事务请求，leader服务器会为其生成对应的事务proposal，并将其发送给集群中所有follower机器，然后收集各自的选票，最后进行事务提交。流程如下图。

 ![img](typora-user-images/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhp1bl80MDgwMzMyOQ==,size_16,color_FFFFFF,t_70) 



ZAB协议的二阶段提交过程中，移除了中断逻辑（事务回滚），所有follower服务器要么正常反馈leader提出的事务proposal，要么就抛弃leader服务器。follower收到proposal后的处理很简单，将该proposal写入到事务日志，然后立马反馈ACK给leader，也就是说如果不是网络、内存或磁盘等问题，follower肯定会写入成功，并正常反馈ACK。leader收到过半follower的ACK后，会广播commit消息给所有learner，并将事务应用到内存；learner收到commit消息后会将事务应用到内存。

 

选主后的数据同步：选主算法中的zxid是从内存数据库中取的最新事务id，事务操作是分两阶段的（提出阶段和提交阶段），leader生成提议并广播给followers，收到半数以上的ACK后，再广播commit消息，同时将事务操作应用到内存中。follower收到提议后先将事务写到本地事务日志，然后反馈ACK，等接到leader的commit消息时，才会将事务操作应用到内存中。可见，选主只是选出了内存数据是最新的节点







 ![img](typora-user-images/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgwMzMy1OQ==,size_16,color_FFFFFF,t_70) 







## 5、ZK集群服务器为什么一般为奇数

•Leader选举算法采用了Paxos协议；

•Paxos核心思想：当多数Server写成功，则任务数据写成功如果有3个Server，则两个写成功即可；如果有4或5个Server，则三个写成功即可。

•Server数目一般为奇数（3、5、7）如果有3个Server，则最多允许1个Server挂掉；如果有4个Server，则同样最多允许1个Server挂掉由此，

 我们看出3台服务器和4台服务器的的容灾能力是一样的，所以为了节省服务器资源，一般我们采用奇数个数，作为服务器部署个数















