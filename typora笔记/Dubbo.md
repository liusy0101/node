[toc]

# Dubbo（分布式服务治理）

## 1、是什么

 Dubbo是一个分布式服务框架，致力于提供高性能和透明化的RPC远程服务调用方案，以及SOA服务治理方案。

核心功能是：

1. 远程通讯: 提供对多种基于长连接的NIO框架抽象封装，包括多种线程模型，序列化，以及“请求-响应”模式的信息交换方式。
2. 集群容错: 提供基于接口方法的透明远程过程调用，包括多协议支持，以及软负载均衡，失败容错，地址路由，动态配置等集群支持。
3. 自动发现: 基于注册中心目录服务，使服务消费方能动态的查找服务提供方，使地址透明，使服务提供方可以平滑增加或减少机器。





## 2、Dubbo架构

### （1）基础架构

![dubbo-architucture](C:\Users\Administrator\Pictures\dubbo-architecture.jpg)



Dubbo系统分为五个部分：远程服务运行容器（Container），远程服务提供方（Provider）、注册中心（Register）、远程服务调用者（Consumer）、监控中心（Monitor）。

Dubbo服务调用过程：

1、服务提供方（Provider）所在的应用在容器中启动并运行（这个容器可以说是该应用部署的tomcat）
2、服务提供方（Provider）将自己要发布的服务注册到注册中心（Registry）
3、服务调用方（Consumer）启动后向注册中心订阅它想要调用的服务
4、注册中心（registry）存储着Provider注册的远程服务，并将其所管理的服务列表通知给服务调用方（Consumer），且

5、注册中心和提供方和调用方之间均保持长连接，可以获取Provider发布的服务的变化情况，并将最新的服务列表推送给Consumer
6、Consumer根据从注册中心获得的服务列表，根据软负载均衡算法选择一个服务提供者（Provider）进行远程服务调用，如果调用失败则选择另一台进行调用。（Consumer会缓存服务列表，即使注册中心宕机也不妨碍进行远程服务调用）
7、监控中心（Monitor）对服务的发布和订阅进行监控和统计服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心（Monitor）;Monitor可以选择Zookeeper、Redis或者Multiast注册中心等，后序介绍。



### （2）架构特点

Dubbo架构具有连通性、健壮性、伸缩性

1>连通性

- 注册中心负责服务地址的注册与查找，相当于目录服务，服务提供者和消费者只在启动时与注册中心交互，注册中心不转发请求，压力较小
- 监控中心负责统计各服务调用次数，调用时间等，统计先在内存汇总后每分钟一次发送到监控中心服务器，并以报表展示
- 服务提供者向注册中心注册其提供的服务，并汇报调用时间到监控中心，此时间不包含网络开销
- 服务消费者向注册中心获取服务提供者地址列表，并根据负载算法直接调用提供者，同时汇报调用时间到监控中心，此时间包含网络开销
- 注册中心，服务提供者，服务消费者三者之间均为长连接，监控中心除外
- 注册中心通过长连接感知服务提供者的存在，服务提供者宕机，注册中心将立即推送事件通知消费者
- 注册中心和监控中心全部宕机，不影响已运行的提供者和消费者，消费者在本地缓存了提供者列表
- 注册中心和监控中心都是可选的，服务消费者可以直连服务提供者

2>健壮性

- 监控中心宕掉不影响使用，只是丢失部分采样数据
- 数据库宕掉后，注册中心仍能通过缓存提供服务列表查询，但不能注册新服务
- 注册中心对等集群，任意一台宕掉后，将自动切换到另一台
- 注册中心全部宕掉后，服务提供者和服务消费者仍能通过本地缓存通讯
- 服务提供者无状态，任意一台宕掉后，不影响使用
- 服务提供者全部宕掉后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复

3>伸缩性

- 注册中心为对等集群，可动态增加机器部署实例，所有客户端将自动发现新的注册中心
- 服务提供者无状态，可动态增加机器部署实例，注册中心将推送新的服务提供者信息给消费者

## 3、使用方法

### （1）服务端

1>创建两个子模块，一个接口模块，一个实现模块

2>接口模块创建接口，实现模块实现接口模块的接口

3>实现模块需要在src下创建一个目录，为MATE-INF，放置log4j.properties（此文件配置之后就可以自动输出日志），在MATE-INF创建一个目录为spring，spring目录下放置provicer.xml配置文件（声明服务接口）。

![image-20200408233908033](.\typora-user-images\image-20200408233908033.png)

4>provider.xml文件下声明服务接口

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://code.alibabatech.com/schema/dubbo
       http://code.alibabatech.com/schema/dubbo/dubbo.xsd http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

    <!--扫描接口实现-->
    <context:component-scan base-package="com.liusy.impl"/>
    <context:annotation-config></context:annotation-config>
    <!-- 提供方应用信息，用于计算依赖关系，name是唯一的-->
    <dubbo:application name="provider"/>

    <!-- 使用zookeeper注册中心暴露服务地址
        id：注册中心唯一id，可以在下面服务注册时使用某个注册中心
        register：是否向此注册中心注册服务，如果设为false，将只订阅其他服务，不把自身进行注册
        subscribe：是否向此注册中心订阅服务，如果设为false，将只将自身进行注册，不订阅其他服务
     -->
    <dubbo:registry id="zk1" file="/home/cache.txt" protocol="zookeeper"
                    address="192.168.197.100:2181,192.168.197.11:2181,192.168.197.120:2181"
                    register="true"
                    subscribe="flase"/>

    <!-- 用dubbo协议在20880端口暴露服务,绑定ip -->
    <dubbo:protocol name="dubbo" port="20880" host="192.168.197.1"/>


    <!-- 声明需要暴露的服务接口  写操作可以设置retries=0 避免重复调用SOA服务
         interface：接口
         ref：实现类
         group：当接口多实现时，可区分具体是哪个实现类
         loadbalance：负载均衡算法
         async：异步调用
        register：需注册到哪一个注册中心，如果缺省，则默认向所有注册中心注册
     -->
    <dubbo:service retries="0" id="SayHelloInterface1"
                   interface="com.liusy.inter.SayHelloInterface"
                   ref="SayHelloImpl"
                   group="1"
                   loadbalance="random"
                   async="false" register="zk1"/>

    <!-- 声明需要暴露的服务接口  写操作可以设置retries=0 避免重复调用SOA服务 -->
    <dubbo:service retries="0" id="SayHelloInterface2" interface="com.liusy.inter.SayHelloInterface" ref="SayHelloImpl2" group="2"/>
</beans>
```

5>启动服务

```java
public static void main( String[] args ) throws IOException {
//        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[]{"simple-provider.xml"});
//        context.start();
//        System.in.read(); // 按任意键退出
        Main.main(args);
    }
```

### （2）客户端

1>引入服务端的接口的jar包

2>需要在src下创建一个目录，为MATE-INF，放置log4j.properties（此文件配置之后就可以自动输出日志），在MATE-INF创建一个目录为spring，spring目录下放置comsumer.xml配置文件（声明服务接口）。

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://code.alibabatech.com/schema/dubbo
       http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <!-- 提供方应用信息，用于计算依赖关系 -->
    <dubbo:application name="client"/>

    <!-- 使用zookeeper注册中心暴露服务地址(zookeeper单节点时，address的值例如：zookeeper://192.168.74.4:2181) -->
    <dubbo:registry address="zookeeper://192.168.197.100:2181?backup=192.168.197.11:2181,192.168.197.120:2181"/>

    <!-- 生成远程服务代理，可以像使用本地bean一样使用demoService 检查级联依赖关系 默认为true 当有依赖服务的时候，需要根据需求进行设置 
     check:如果check为false，表示启动的时候不去检查。当服务出现循环依赖的时候，check设置成false
                               -->
    <dubbo:reference id="SayHelloInterface"  check="false" interface="com.liusy.inter.SayHelloInterface" group="1"  async="false">
        <dubbo:method name="sayHello">
        </dubbo:method>
    </dubbo:reference>


</beans>
```

 3>调用服务

``` java
public static void main( String[] args ) throws IOException {
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[] {"META-INF.spring/simple-consumer.xml"});
    context.start();

    SayHelloInterface simpleService = (SayHelloInterface) context.getBean("SayHelloInterface");
    String helloResult = simpleService.sayHello();
    System.out.println("sayHello result:"+helloResult);

    System.out.println("===============================");

    System.in.read();
}
```

 也可以集成spring直接注入。

## 4、负载均衡

在集群负载均衡时，Dubbo提供了多种均衡策略，缺省为random随机调用。可以自行扩展负载均衡策略

### （1）Random LoadBalance

随机，按权重设置随机概率。
在一个截面上碰撞的概率高，但调用量越大分布越均匀，而且按概率使用权重后也比较均匀，有利于动态调整提供者权重。

### （2）RoundRobin LoadBalance

轮循，按公约后的权重设置轮循比率。
存在慢的提供者累积请求的问题，比如：第二台机器很慢，但没挂，当请求调到第二台时就卡在那，久而久之，所有请求都卡在调到第二台上。

### （3）LeastActive LoadBalance

最少活跃调用数，相同活跃数的随机，活跃数指调用前后计数差。
使慢的提供者收到更少请求，因为越慢的提供者的调用前后计数差会越大。

### （4）ConsistentHash LoadBalance

一致性Hash，相同参数的请求总是发到同一提供者。
当某一台提供者挂时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动。



