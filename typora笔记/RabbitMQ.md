[toc]

## RabbitMQ

### 一、安装：

#### 1、安装Erlang

##### （1）先获取包

wget http://www.rabbitmq.com/releases/erlang/erlang-19.0.4-1.el7.centos.x86_64.rpm

##### （2）利用rpm进行安装

rpm -ivh erlang-19.0.4-1.el7.centos.x86_64.rpm

##### （3）查看是否安装成功

erl -version

#### 2、安装rabbitmq

##### （1）先获取包

wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.7.15/rabbitmq-server-3.7.15-1.el7.noarch.rpm

##### （2）安装

rpm -ivh rabbitmq-server-3.4.1-1.noarch.rpm

##### （3）设置开机启动

chkconfig rabbitmq-server on

###### （4）设置配置文件

cd /etc/rabbitmq

cp /usr/share/doc/rabbitmq-server-3.4.1/rabbitmq.config.example /etc/rabbitmq/

mv rabbitmq.config.example rabbitmq.config

##### （5）开启用户远程访问

![image-20200712170228637](.\typora-user-images\image-20200712170228637.png)

把前面的“%%”和最后面的“，”去掉

##### （6）开启web界面管理工具

rabbitmq-plugins enable rabbitmq_management

开启之后访问15672端口，如果有开启防火墙，需要开放此端口

/sbin/iptables -I INPUT -p tcp --dport 15672 -j ACCEPT

/etc/rc.d/init.d/iptables save

![image-20200712170440384](.\typora-user-images\image-20200712170440384.png)

默认用户名和密码都是guest

![image-20200712170708071](.\typora-user-images\image-20200712170708071.png)

##### （7）启停命令

service rabbitmq-server start

service rabbitmq-server stop

service rabbitmq-server restart

### 二、角色以及Virtual Host配置

#### 1、角色

![image-20200712170940703](.\typora-user-images\image-20200712170940703.png)

可在此页面添加用户，同时配置相应的角色，RabbitMQ提供了五种角色，分别是：

#### （1）超级管理员(administrator)

可登陆管理控制台，可查看所有的信息，并且可以对用户，策略(policy)进行操作。

#### （2）监控者(monitoring)

可登陆管理控制台，同时可以查看rabbitmq节点的相关信息(进程数，内存使用情况，磁盘使用情况等)

#### （3）策略制定者(policymaker)

可登陆管理控制台, 同时可以对policy进行管理。但无法查看节点的相关信息(上图红框标识的部分)。

#### （4）普通管理者(management)

仅可登陆管理控制台，无法看到节点信息，也无法对策略进行管理。

#### （5）其他

无法登陆管理控制台，通常就是普通的生产者和消费者。

#### 2、Virtual Hosts配置

像mysql拥有数据库的概念并且可以指定用户对库和表等操作的权限。RabbitMQ也有类似的权限管理；在RabbitMQ中可以虚拟消息服务器Virtual Host，每个Virtual Hosts相当于一个相对独立的RabbitMQ服务器，每个VirtualHost之间是相互隔离的。exchange、queue、message不能互通。 相当于mysql的db。Virtual Name一般以/开头。

##### （1）创建Virtual Host

![image-20200712171558203](.\typora-user-images\image-20200712171558203.png)

##### （2）设置Virtual Host权限

1、点击相应的Virtual Host

![image-20200712171728356](.\typora-user-images\image-20200712171728356.png)

2、设置访问权限

![image-20200712171749619](.\typora-user-images\image-20200712171749619.png)

1. user：用户名
2. configure ：一个正则表达式，用户对符合该正则表达式的所有资源拥有 configure 操作的权限
3. write：一个正则表达式，用户对符合该正则表达式的所有资源拥有 write 操作的权限
4. read：一个正则表达式，用户对符合该正则表达式的所有资源拥有 read 操作的权限



### 三、Work queues工作队列模式

一个队列对应一个或多个消费者，共同消费一个队列里的消息，一个消息只能一个消费者消费。

**应用场景**：对于任务过重或任务较多情况，使用工作队列模式使用多个消费者可以提高任务处理的速度。



### 四、Publish/Subscribe发布订阅模式

![image-20200712181203906](.\typora-user-images\image-20200712181203906.png)

在发布订阅模型中，多了一个x(exchange)角色，而且过程略有变化。

P：生产者，也就是要发送消息的程序，但是不再发送到队列中，而是发给X（交换机）
C：消费者，消息的接受者，会一直等待消息到来。
Queue：消息队列，接收消息、缓存消息。
Exchange：交换机，图中的X。一方面，接收生产者发送的消息。另一方面，知道如何处理消息，例如递交给某个特别队列、递交给所有队列、或是将消息丢弃。到底如何操作，取决于Exchange的类型。Exchange有常见以下3种类型：
	Fanout：广播，将消息交给所有绑定到交换机的队列
	Direct：定向，把消息交给符合指定routing key 的队列
	Topic：通配符，把消息交给符合routing pattern（路由模式） 的队列

***\*Exchange（交换机）只负责转发消息，不具备存储消息的能力\****，因此如果没有任何队列与Exchange***\*绑定\****，或者没有符合路由规则的队列，那么消息会丢失！

![image-20200712181317902](.\typora-user-images\image-20200712181317902.png)

交换机需要与队列进行绑定，绑定之后；一个消息可以被多个消费者都收到。
发布订阅模式与work队列模式的区别
1、work队列模式不用定义交换机，而发布/订阅模式需要定义交换机。 
2、发布/订阅模式的生产方是面向交换机发送消息，work队列模式的生产方是面向队列发送消息(底层使用默认交换机)。
3、发布/订阅模式的消费者需要设置队列和交换机的绑定，work队列模式不需要设置，实际上work队列模式会将队列绑 定到默认的交换机 。



### 五、Routing模式

路由模式特点：

1.队列与交换机的绑定，不能是任意绑定了，而是要指定一个RoutingKey（路由key）

2.消息的发送方在 向 Exchange发送消息时，也必须指定消息的 RoutingKey。

3.Exchange不再把消息交给每一个绑定的队列，而是根据消息的Routing Key进行判断，只有队列的Routingkey与消息的 Routing key完全一致，才会接收到消息

![image-20200712183024299](.\typora-user-images\image-20200712183024299.png)

P：生产者，向Exchange发送消息，发送消息时，会指定一个routing key。

X：Exchange（交换机），接收生产者的消息，然后把消息递交给 与routing key完全匹配的队列

C1：消费者，其所在队列指定了需要routing key 为 error 的消息

C2：消费者，其所在队列指定了需要routing key 为 info、error、warning 的消息





Routing模式要求队列在绑定交换机时要指定routing key，消息会转发到符合routing key的队列。





### 六、Topic模式

![image-20200712184445526](.\typora-user-images\image-20200712184445526.png)

Topic类型与Direct相比，都是可以根据RoutingKey把消息路由到不同的队列。只不过Topic类型Exchange可以让队列在绑定Routing key的时候使用通配符！

Routingkey 一般都是有一个或多个单词组成，多个单词之间以“ . ”分割，例如： item.insert

 

通配符规则：

\#：匹配一个或多个词

*：匹配不多不少恰好1个词

 

举例：

item.#：能够匹配item.insert.abc 或者 item.insert

item.*：只能匹配item.insert



![img](file:///C:\Users\ADMINI~1\AppData\Local\Temp\1\ksohtml1156\wps1.jpg) 

图解：

红色Queue：绑定的是usa.# ，因此凡是以 usa.开头的routing key 都会被匹配到

黄色Queue：绑定的是#.news ，因此凡是以 .news结尾的 routing key 都会被匹配





### 六、模式总结

RabbitMQ工作模式：

***\*1、简单模式 HelloWorld\****

一个生产者、一个消费者，不需要设置交换机（使用默认的交换机）

***\*2、工作队列模式 Work Queue\****

一个生产者、多个消费者（竞争关系），不需要设置交换机（使用默认的交换机）

***\*3、发布订阅模式 Publish/subscribe\****

需要设置类型为fanout的交换机，并且交换机和队列进行绑定，当发送消息到交换机后，交换机会将消息发送到绑定的队列

***\*4、路由模式 Routing\****

需要设置类型为direct的交换机，交换机和队列进行绑定，并且指定routing key，当发送消息到交换机后，交换机会根据routing key将消息发送到对应的队列

***\*5、通配符模式 Topic\****

需要设置类型为topic的交换机，交换机和队列进行绑定，并且指定通配符方式的routing key，当发送消息到交换机后，交换机会根据routing key将消息发送到对应的队列



### 七、问题

#### 1、消息如何分发？

若该队列至少有一个消费者订阅，消息将以循环（round-robin）的方式发送给消费者。每条消息只会分发给一个订阅的消费者（前提是消费者能够正常处理消息并进行确认）。

#### 2、如何确保消息正确地发送至RabbitMQ？

RabbitMQ使用发送方确认模式，确保消息正确地发送到RabbitMQ。发送方确认模式：将信道设置成confirm模式（发送方确认模式），则所有在信道上发布的消息都会被指派一个唯一的ID。一旦消息被投递到目的队列后，或者消息被写入磁盘后（可持久化的消息），信道会发送一个确认给生产者（包含消息唯一ID）。如果RabbitMQ发生内部错误从而导致消息丢失，会发送一条nack（not acknowledged，未确认）消息。发送方确认模式是异步的，生产者应用程序在等待确认的同时，可以继续发送消息。当确认消息到达生产者应用程序，生产者应用程序的回调方法就会被触发来处理确认消息。

#### 3、如何确保消息接收方消费了消息？

接收方消息确认机制：消费者接收每一条消息后都必须进行确认（消息接收和消息确认是两个不同操作）。只有消费者确认了消息，RabbitMQ才能安全地把消息从队列中删除。这里并没有用到超时机制，RabbitMQ仅通过Consumer的连接中断来确认是否需要重新发送消息。也就是说，只要连接不中断，RabbitMQ给了Consumer足够长的时间来处理消息。

下面罗列几种特殊情况：

如果消费者接收到消息，在确认之前断开了连接或取消订阅，RabbitMQ会认为消息没有被分发，然后重新分发给下一个订阅的消费者。（可能存在消息重复消费的隐患，需要根据bizId去重）
如果消费者接收到消息却没有确认消息，连接也未断开，则RabbitMQ认为该消费者繁忙，将不会给该消费者分发更多的消息。

#### 4、如何避免消息重复投递或重复消费？

在消息生产时，MQ内部针对每条生产者发送的消息生成一个inner-msg-id，作为去重和幂等的依据（消息投递失败并重传），避免重复的消息进入队列；在消息消费时，要求消息体中必须要有一个bizId（对于同一业务全局唯一，如支付ID、订单ID、帖子ID等）作为去重和幂等的依据，避免同一条消息被重复消费。

这个问题针对业务场景来答分以下几点：

1.比如，你拿到这个消息做数据库的insert操作。那就容易了，给这个消息做一个唯一主键，那么就算出现重复消费的情况，就会导致主键冲突，避免数据库出现脏数据。

2.再比如，你拿到这个消息做redis的set的操作，那就容易了，不用解决，因为你无论set几次结果都是一样的，set操作本来就算幂等操作。

3.如果上面两种情况还不行，上大招。准备一个第三方介质,来做消费记录。以redis为例，给消息分配一个全局id，只要消费过该消息，将<id,message>以K-V形式写入redis。那消费者开始消费前，先去redis中查询有没消费记录即可。

#### 5、如何解决丢数据的问题?

##### 1.生产者丢数据

生产者的消息没有投递到MQ中怎么办？从生产者弄丢数据这个角度来看，RabbitMQ提供transaction和confirm模式来确保生产者不丢消息。

transaction机制就是说，发送消息前，开启事物(channel.txSelect())，然后发送消息，如果发送过程中出现什么异常，事物就会回滚(channel.txRollback())，如果发送成功则提交事物(channel.txCommit())。

然而缺点就是吞吐量下降了。因此，按照博主的经验，生产上用confirm模式的居多。一旦channel进入confirm模式，所有在该信道上面发布的消息都将会被指派一个唯一的ID(从1开始)，一旦消息被投递到所有匹配的队列之后，rabbitMQ就会发送一个Ack给生产者(包含消息的唯一ID)，这就使得生产者知道消息已经正确到达目的队列了.如果rabiitMQ没能处理该消息，则会发送一个Nack消息给你，你可以进行重试操作。

##### 2.消息队列丢数据

处理消息队列丢数据的情况，一般是开启持久化磁盘的配置。这个持久化配置可以和confirm机制配合使用，你可以在消息持久化磁盘后，再给生产者发送一个Ack信号。这样，如果消息持久化磁盘之前，rabbitMQ阵亡了，那么生产者收不到Ack信号，生产者会自动重发。

那么如何持久化呢，这里顺便说一下吧，其实也很容易，就下面两步

①、将queue的持久化标识durable设置为true,则代表是一个持久的队列

②、发送消息的时候将deliveryMode=2

这样设置以后，rabbitMQ就算挂了，重启后也能恢复数据。在消息还没有持久化到硬盘时，可能服务已经死掉，这种情况可以通过引入mirrored-queue即镜像队列，但也不能保证消息百分百不丢失（整个集群都挂掉）

##### 3.消费者丢数据

启用手动确认模式可以解决这个问题

①自动确认模式，消费者挂掉，待ack的消息回归到队列中。消费者抛出异常，消息会不断的被重发，直到处理成功。不会丢失消息，即便服务挂掉，没有处理完成的消息会重回队列，但是异常会让消息不断重试。

②手动确认模式，如果消费者来不及处理就死掉时，没有响应ack时会重复发送一条信息给其他消费者；如果监听程序处理异常了，且未对异常进行捕获，会一直重复接收消息，然后一直抛异常；如果对异常进行了捕获，但是没有在finally里ack，也会一直重复发送消息(重试机制)。

③不确认模式，acknowledge="none" 不使用确认机制，只要消息发送完成会立即在队列移除，无论客户端异常还是断开，只要发送完就移除，不会重发。



#### 6、死信队列和延迟队列的使用

##### 1、死信消息：

消息被拒绝（Basic.Reject或Basic.Nack）并且设置 requeue 参数的值为 false
消息过期了
队列达到最大的长度

##### 2、过期消息：

    在 rabbitmq 中存在2种方可设置消息的过期时间，第一种通过对队列进行设置，这种设置后，该队列中所有的消息都存在相同的过期时间，第二种通过对消息本身进行设置，那么每条消息的过期时间都不一样。如果同时使用这2种方法，那么以过期时间小的那个数值为准。当消息达到过期时间还没有被消费，那么那个消息就成为了一个 死信 消息。
    
    队列设置：在队列申明的时候使用 x-message-ttl 参数，单位为 毫秒
    
    单个消息设置：是设置消息属性的 expiration 参数的值，单位为 毫秒

延时队列：在rabbitmq中不存在延时队列，但是我们可以通过设置消息的过期时间和死信队列来模拟出延时队列。消费者监听死信交换器绑定的队列，而不要监听消息发送的队列。

Map<String, Object> arguments = new HashMap<String, Object>(); 
// 统一设置队列中的所有消息的过期时间 
arguments.put("x-message-ttl", 30000); 
// 设置超过多少毫秒没有消费者来访问队列，就删除队列的时间 
arguments.put("x-expires", 20000); 
// 设置队列的最新的N条消息，如果超过N条，前面的消息将从队列中移除掉 
arguments.put("x-max-length", 4); 
// 设置队列的内容的最大空间，超过该阈值就删除之前的消息
arguments.put("x-max-length-bytes", 1024); 
// 将删除的消息推送到指定的交换机，一般x-dead-letter-exchange和x-dead-letter-routing-key需要同时设置
arguments.put("x-dead-letter-exchange", "exchange.dead"); 
// 将删除的消息推送到指定的交换机对应的路由键 
arguments.put("x-dead-letter-routing-key", "routingkey.dead"); 
// 设置消息的优先级，优先级大的优先被消费 
arguments.put("x-max-priority", 10);



### 八、消息积压解决方案

在设计系统的时候，一定要保证消费端的消费性能要高于生产端的发送性能

导致积压突然增加，最粗粒度的原因，只有两种：要么是发送变快了，要么是消费变慢了

消息积压处理：
1、发送端优化，增加批量和线程并发两种方式处理
2、消费端优化，优化业务逻辑代码、水平扩容增加并发并同步扩容分区数量

如果短时间内没有足够的服务器资源进行扩容，只能将系统降级，通过关闭一些不重要的哑无，减少发送方发送的数据量，最低限度保障系统正常运转。

查看消息积压的方法：
1、消息队列内置监控，查看发送端发送消息与消费端消费消息的速度变化
2、查看日志是否有大量的消费错误
3、打印堆栈信息，查看消费线程卡点信息




