# 个人日记

## 一、Gradle

### 1、启动报错CreateProcess error=206, 文件名或扩展名太长。

![1595386162995](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1595386162995.png)

只需在项目中的build.gradle中加上：

```groovy
buildscript {
    repositories {
        maven {
            url "https://plugins.gradle.org/m2/"
        }
    }
    dependencies {
        classpath "gradle.plugin.ua.eshepelyuk:ManifestClasspath:1.0.0"
    }
}

apply plugin: "ua.eshepelyuk.ManifestClasspath"
```



## 二、在服务器上反编译查看某个类

```shell
cd arthas 
java -jar arthas-boot.jar

jad
反编译某个类
jad  jdy.bos.login.util.AccountStatusUtil
```





## 三、Arthas

中文文档： https://alibaba.github.io/arthas/install-detail.html 

### 1、是什么

Arthas是Alibaba开源的java诊断工具。

### 2、可以解决什么？

（1）这个类从哪个jar包加载的？为什么会报各种类相关的Exception。

（2）更改的代码为什么没执行？

（3）无法线上debug的问题

（4）线上某个数据有问题，但无法线上debug，线下无法重现。

（5）是否有一个全局视角来查看系统的运行状况？

（6）有什么办法可以监控到JVM的实时运行状态？

### 3、基本使用

#### （1）启动arthas-boot

下载arthas-boot.jar，再用java -jar启动

```shell
wget https://alibaba.github.io/arthas/arthas-boot.jar
java -jar arthas-boot.jar --target-ip 0.0.0.0 -h
```

-h：表示输出帮助信息

arthas-boot.jar是Arthas的启动程序，启动后，会列出所有的java进程。

![1596594166393](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596594166393.png)

如果想跟进某个java进程，只需要输入对应的序号即可，Arthas会attach到目标进程上，并输出日志。![1596594280041](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596594280041.png)

#### （2）查看dashboard

输入dashboard，会展示当前进程的信息

#### （3）通过sysenv命令获取进程的Main class

```shell
$ sysenv | grep MAIN
 JAVA_MAIN_CLASS_71560              demo.MathGame
```

#### （4）通过jad来反编译某个类

```shell
jad 类的全限定名
```

#### （5）watch

通过watch命令来查看某个类的某个方法的返回值

```shell
watch 类的全限定名 方法名 returnObj
```

例如：

```shell
watch demo.MathGame primeFactors returnObj
```



### 4、进阶使用

#### 基础命令

help——查看命令帮助信息
cls——清空当前屏幕区域
session——查看当前会话的信息
reset——重置增强类，将被 Arthas 增强过的类全部还原，Arthas 服务端关闭时会重置所有增强过的类
version——输出当前目标 Java 进程所加载的 Arthas 版本号
history——打印命令历史
quit——退出当前 Arthas 客户端，其他 Arthas 客户端不受影响
shutdown——关闭 Arthas 服务端，所有 Arthas 客户端全部退出
keymap——Arthas快捷键列表及自定义快捷键

#### jvm相关

dashboard——当前系统的实时数据面板
thread——查看当前 JVM 的线程堆栈信息
jvm——查看当前 JVM 的信息
sysprop——查看和修改JVM的系统属性
sysenv——查看JVM的环境变量
New!getstatic——查看类的静态属性

#### class/classloader相关

sc——查看JVM已加载的类信息
sm——查看已加载类的方法信息
dump——dump 已加载类的 byte code 到特定目录
redefine——加载外部的.class文件，redefine到JVM里
jad——反编译指定已加载类的源码
classloader——查看classloader的继承树，urls，类加载信息，使用classloader去getResource

#### monitor/watch/trace相关

请注意，这些命令，都通过字节码增强技术来实现的，会在指定类的方法中插入一些切面来实现数据统计和观测，因此在线上、预发使用时，请尽量明确需要观测的类、方法以及条件，诊断结束要执行 shutdown 或将增强过的类执行 reset 命令。
monitor——方法执行监控
watch——方法执行数据观测
trace——方法内部调用路径，并输出方法路径上的每个节点上耗时
stack——输出当前方法被调用的调用路径
tt——方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测

#### options

options——查看或设置Arthas全局开关

#### 管道

Arthas支持使用管道对上述命令的结果进行进一步的处理，如sm org.apache.log4j.Logger | grep <init>

grep——搜索满足条件的结果
plaintext——将命令的结果去除颜色
wc——按行统计输出结果

#### 后台异步任务

当线上出现偶发的问题，比如需要watch某个条件，而这个条件一天可能才会出现一次时，异步后台任务就派上用场了，详情请参考这里

使用 > 将结果重写向到日志文件，使用 & 指定命令是后台运行，session断开不影响任务执行（生命周期默认为1天）
jobs——列出所有job
kill——强制终止任务
fg——将暂停的任务拉到前台执行
bg——将暂停的任务放到后台执行

#### Web Console

通过websocket连接Arthas。

Web Console

#### 其他特性

异步命令支持
执行结果存日志
批处理的支持
ognl表达式的用法说明



### 5、命令详细

#### （1）dashboard

当前进程的实时数据面板（包括内存，线程等信息），按ctrl+c退出。

数据说明：

```
ID: Java级别的线程ID，注意这个ID不能跟jstack中的nativeID一一对应
NAME: 线程名
GROUP: 线程组名
PRIORITY: 线程优先级, 1~10之间的数字，越大表示优先级越高
STATE: 线程的状态
CPU%: 线程消耗的cpu占比，采样100ms，将所有线程在这100ms内的cpu使用量求和，再算出每个线程的cpu使用占比。
TIME: 线程运行总时间，数据格式为分：秒
INTERRUPTED: 线程当前的中断位状态
DAEMON: 是否是daemon线程
```

#### （2）Thread

查看当前线程信息，查看线程的堆栈。

参数说明：

```
id	线程id
[n:]	指定最忙的前N个线程并打印堆栈
[b]	找出当前阻塞其他线程的线程
[i <value>]	指定cpu占比统计的采样间隔，单位为毫秒
```

1、一键展示当前最忙的前n个线程并打印堆栈。

```
thread -n 3
```

2、不带参数时，显示所有线程的信息

3、显示指定线程的运行堆栈

```
therad id
```

4、找出当前阻塞其他线程的线程

```
thread -b
```

  目前只支持找出synchronized关键字阻塞住的线程， 如果是`java.util.concurrent.Lock`， 目前还不支持 

5、指定采样时间间隔

```
thread -i 时间
```

例如：thread -n 3 -i 1000

#### （3）jvm

查看当前JVM信息

信息说明：

```
THREAD相关
COUNT: JVM当前活跃的线程数
DAEMON-COUNT: JVM当前活跃的守护线程数
PEAK-COUNT: 从JVM启动开始曾经活着的最大线程数
STARTED-COUNT: 从JVM启动开始总共启动过的线程次数
DEADLOCK-COUNT: JVM当前死锁的线程数

文件描述符相关
MAX-FILE-DESCRIPTOR-COUNT：JVM进程最大可以打开的文件描述符数
OPEN-FILE-DESCRIPTOR-COUNT：JVM当前打开的文件描述符数
```

#### （4）sysprop

查看当前jvm的系统属性（system property）

1、查看所有属性

```
sysprop
```

2、查看某个属性

```
sysprop java.version
```

3、修改某个属性值

```
sysprop key value
```

#### （5）sysenv

查看当前jvm的环境属性（system environment variables）

1、查看所有环境变量

```
sysenv
```

2、查看单个环境变量

```
sysenv USER
```

#### （6）getstatic

可以查看类的静态属性

```
getstatic class_name field_name
```

如果该静态属性是一个复杂对象，还可以支持在该属性上通过ognl表示进行遍历，过滤，访问对象的内部属性等操作。

例如，假设n是一个Map，Map的Key是一个Enum，我们想过滤出Map中Key为某个Enum的值，可以写如下命令

```shell
$ getstatic com.alibaba.arthas.Test n 'entrySet().iterator.{? #this.key.name()=="STOP"}'
field: n
@ArrayList[
    @Node[STOP=bbb],
]
```

#### （7）ognl

执行ognl表达式

参数说明：

```
express	执行的表达式
[c:]	执行表达式的 ClassLoader 的 hashcode，默认值是SystemClassLoader
[x]	    结果对象的展开层次，默认值1
```

1、调用静态函数

```
ognl '@java.lang.System@out.println("hello")'
```

2、获取静态类的静态字段

```
ognl '@demo.MathGame@random'
```

3、执行多行表达式，赋值给临时变量，返回一个list

```
$ ognl '#value1=@System@getProperty("java.home"), #value2=@System@getProperty("java.runtime.name"), {#value1, #value2}'
@ArrayList[
    @String[/opt/java/8.0.181-zulu/jre],
    @String[OpenJDK Runtime Environment],
]
```

#### （8）sc

查看jvm已加载的类信息

 Search-Class” 的简写，这个命令能搜索出所有已经加载到 JVM 中的 Class 信息，这个命令支持的参数有 `[d]`、`[E]`、`[f]` 和 `[x:]`。 

参数说明：

```
class-pattern	类名表达式匹配
method-pattern	方法名表达式匹配
[d]	输出当前类的详细信息，包括这个类所加载的原始文件来源、类的声明、加载的ClassLoader等详细信息。如果一个类被多个ClassLoader所加载，则会出现多次
[E]	开启正则表达式匹配，默认为通配符匹配
[f]	输出当前类的成员变量信息（需要配合参数-d一起使用）
[x:]	指定输出静态变量时属性的遍历深度，默认为 0，即直接使用 toString 输出
```

> class-pattern支持全限定名，如com.taobao.test.AAA，也支持com/taobao/test/AAA这样的格式，这样，我们从异常堆栈里面把类名拷贝过来的时候，不需要在手动把`/`替换为`.`啦。

> sc 默认开启了子类匹配功能，也就是说所有当前类的子类也会被搜索出来，想要精确的匹配，请打开`options disable-sub-class true`开关

 1、查看类的静态变量信息 

```
sc -df 类的全限定名
```



#### （9）sm

查看已加载类的方法信息

“Search-Method” 的简写，这个命令能搜索出所有已经加载了 Class 信息的方法信息。

`sm` 命令只能看到由当前类所声明 (declaring) 的方法，父类则无法看到。

```
class-pattern	类名表达式匹配
method-pattern	方法名表达式匹配
[d]	展示每个方法的详细信息
[E]	开启正则表达式匹配，默认为通配符匹配
```

#### （10）dump

dump已加载类的bytecode到特定目录

参数说明：

```
class-pattern	类名表达式匹配
[c:]	类所属 ClassLoader 的 hashcode
[E]	开启正则表达式匹配，默认为通配符匹配
```

#### （11）jad

反编译已加载类的源码

`jad` 命令将 JVM 中实际运行的 class 的 byte code 反编译成 java 代码，便于你理解业务逻辑；

- 在 Arthas Console 上，反编译出来的源码是带语法高亮的，阅读更方便
- 当然，反编译出来的 java 代码可能会存在语法错误，但不影响你进行阅读理解

```shell
class-pattern	类名表达式匹配
[c:]	类所属 ClassLoader 的 hashcode
[E]	开启正则表达式匹配，默认为通配符匹配
```

1、反编译指定方法

```
jad 类的全限定名 方法
```

#### （12）classloader

查看classloader的继承树，urls，类加载信息

`classloader` 命令将 JVM 中所有的classloader的信息统计出来，并可以展示继承树，urls等。

可以让指定的classloader去getResources，打印出所有查找到的resources的url。对于`ResourceNotFoundException`比较有用。

参数说明：

```
参数名称	参数说明
[l]	按类加载实例进行统计
[t]	打印所有ClassLoader的继承树
[a]	列出所有ClassLoader加载的类，请谨慎使用
[c:]	ClassLoader的hashcode
[c: r:]	用ClassLoader去查找resource
```

1、按类加载类型查看统计信息

```
classloader
```

2、按类加载实例查看统计信息

```
classloader -l
```

3、查看ClassLoader的继承树

```
classloader -t
```

4、查看URLClassLoader实际的urls

```
classloader -c 5ffe9775
```

5、使用ClassLoader去查找resource

```
$ classloader -c 226b143b -r META-INF/MANIFEST.MF
```

6、 查找类的class文件： 

```
$ classloader -c 1b6d3586 -r java/lang/String.class
```

#### （13）redefine

加载外部的.class文件，redefine jvm已加载的类。

```
[c:]	ClassLoader的hashcode
[p:]	外部的.class文件的完整路径，支持多个
```

```
例如：redefine -p /tmp/Test.class
```

#### （14）monitor

方法执行监控

对匹配 `class-pattern`／`method-pattern`的类、方法的调用进行监控。

`monitor` 命令是一个非实时返回命令.

实时返回命令是输入之后立即返回，而非实时返回的命令，则是不断的等待目标 Java 进程返回信息，直到用户输入 `Ctrl+C` 为止。

服务端是以任务的形式在后台跑任务，植入的代码随着任务的中止而不会被执行，所以任务关闭后，不会对原有性能产生太大影响，而且原则上，任何Arthas命令不会引起原有业务逻辑的改变。

监控维度：

```
timestamp	时间戳
class	Java类
method	方法（构造方法、普通方法）
total	调用次数
success	成功次数
fail	失败次数
rt	平均RT
fail-rate	失败率
```

参数：

```
class-pattern	类名表达式匹配
method-pattern	方法名表达式匹配
[E]	开启正则表达式匹配，默认为通配符匹配
[c:]	统计周期，默认值为120秒
```

例如：

```
$ monitor -c 5 com.alibaba.sample.petstore.web.store.module.screen.ItemList execute
```

#### （15）watch

方法执行数据监测

 让你能方便的观察到指定方法的调用情况。能观察到的范围为：`返回值`、`抛出异常`、`入参`，通过编写 OGNL 表达式进行对应变量的查看。 

参数：

```
class-pattern	类名表达式匹配
method-pattern	方法名表达式匹配
express	观察表达式
condition-express	条件表达式
[b]	在方法调用之前观察
[e]	在方法异常之后观察
[s]	在方法返回之后观察
[f]	在方法结束之后(正常返回和异常返回)观察
[E]	开启正则表达式匹配，默认为通配符匹配
[x:]	指定输出结果的属性遍历深度，默认为 1
```

 这里重点要说明的是观察表达式，观察表达式的构成主要由 ognl 表达式组成，所以你可以这样写`"{params,returnObj}"`，只要是一个合法的 ognl 表达式，都能被正常支持。 

1、观察方法出参和返回值

```
$ watch com.alibaba.sample.petstore.web.store.module.screen.ItemList add "{params,returnObj}" -x 2
```

2、观察方法入参

```
$ watch com.alibaba.sample.petstore.web.store.module.screen.ItemList add "{params,returnObj}" -x 2 -b
```

3、同时观察方法调用前和方法返回后

```
$ watch com.alibaba.sample.petstore.web.store.module.screen.ItemList add "{params,returnObj}" -x 2 -b -s 
```

4、调整-x的值，观察具体的方法参数值

```
$ watch com.alibaba.sample.petstore.web.store.module.screen.ItemList add "{params,returnObj}" -x 3 
```

5、条件表达式的例子

```
$ watch UserManagerImpl testAdd "{params, returnObj}" "params[0].equals('aaa')" -x 2 
```

 只有满足条件的调用，才会有响应 

6、观察异常信息

```
$ watch com.alibaba.sample.petstore.biz.impl.UserManagerImpl testAdd "{params, throwExp}"  -e -x 2 
```

7、按照耗时进行过滤

```
$ watch com.alibaba.sample.petstore.web.store.module.screen.ItemList add "{params,returnObj}" #cost>200 -x 3 
```

> \#cost>200(单位是`ms`)表示只有当耗时大于200ms时才会输出，过滤掉执行时间小于200ms的调用

8、观察当前对象中的全局属性

```
$ watch com.taobao.container.web.arthas.rest.MyAppsController myFavoriteApps 'target'
```

 如果想查看方法运行前后，当前对象中的全局属性，可以使用`target`关键字，代表当前对象 

然后使用`target.field_name`访问当前对象的某个全局属性

```
$ watch com.taobao.container.web.arthas.rest.MyAppsController myFavoriteApps 'target.myFavAppsMapper'
```

#### （16）trace

方法内部调用路径，并输出方法路径上的每个节点上耗时

 `trace` 命令能主动搜索 `class-pattern`／`method-pattern` 对应的方法调用路径，渲染和统计整个调用链路上的所有性能开销和追踪调用链路。 

```
class-pattern	类名表达式匹配
method-pattern	方法名表达式匹配
condition-express	条件表达式
[E]	开启正则表达式匹配，默认为通配符匹配
[n:]	命令执行次数
#cost	方法执行耗时
```

例如：

```
trace com.alibaba.sample.petstore.web.store.module.screen.ItemList add params.length==2
```



按照耗时过滤：

```
trace com.alibaba.sample.petstore.web.store.module.screen.ItemList execute #cost>4
```



#### （17）stack

输出当前方法被调用的调用路径

 很多时候我们都知道一个方法被执行，但这个方法被执行的路径非常多，或者你根本就不知道这个方法是从那里被执行了，此时你需要的是 stack 命令。 

```
class-pattern	类名表达式匹配
method-pattern	方法名表达式匹配
condition-express	条件表达式
[E]	开启正则表达式匹配，默认为通配符匹配
[n:]	执行次数限制
```

例如：

按照耗时查询，只会打印出耗时超过30ms的堆栈情况 

```
stack com.alibaba.sample.petstore.web.store.module.screen.ItemList execute #cost>30
```



#### （18）tt

方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测

`watch` 虽然很方便和灵活，但需要提前想清楚观察表达式的拼写，这对排查问题而言要求太高，因为很多时候我们并不清楚问题出自于何方，只能靠蛛丝马迹进行猜测。

这个时候如果能记录下当时方法调用的所有入参和返回值、抛出的异常会对整个问题的思考与判断非常有帮助。

于是乎，TimeTunnel 命令就诞生了。

- 命令参数解析
  - `-t`

tt 命令有很多个主参数，`-t` 就是其中之一。这个参数的表明希望记录下类 `*Test` 的 `print` 方法的每次执行情况。

- `-n 3`

当你执行一个调用量不高的方法时可能你还能有足够的时间用 `CTRL+C` 中断 tt 命令记录的过程，但如果遇到调用量非常大的方法，瞬间就能将你的 JVM 内存撑爆。

此时你可以通过 `-n` 参数指定你需要记录的次数，当达到记录次数时 Arthas 会主动中断tt命令的记录过程，避免人工操作无法停止的情况。

```
INDEX	时间片段记录编号，每一个编号代表着一次调用，后续tt还有很多命令都是基于此编号指定记录操作，非常重要。
TIMESTAMP	方法执行的本机时间，记录了这个时间片段所发生的本机时间
COST(ms)	方法执行的耗时
IS-RET	方法是否以正常返回的形式结束
IS-EXP	方法是否以抛异常的形式结束
OBJECT	执行对象的hashCode()，注意，曾经有人误认为是对象在JVM中的内存地址，但很遗憾他不是。但他能帮助你简单的标记当前执行方法的类实体
CLASS	执行的类名
METHOD	执行的方法名
```



#### （19）options

全局开关

| 名称               | 默认值 | 描述                                                         |
| :----------------- | :----- | :----------------------------------------------------------- |
| unsafe             | false  | 是否支持对系统级别的类进行增强，打开该开关可能导致把JVM搞挂，请慎重选择！ |
| dump               | false  | 是否支持被增强了的类dump到外部文件中，如果打开开关，class文件会被dump到`/${application dir}/arthas-class-dump/`目录下，具体位置详见控制台输出 |
| batch-re-transform | true   | 是否支持批量对匹配到的类执行retransform操作                  |
| json-format        | false  | 是否支持json化的输出                                         |
| disable-sub-class  | false  | 是否禁用子类匹配，默认在匹配目标类的时候会默认匹配到其子类，如果想精确匹配，可以关闭此开关 |
| debug-for-asm      | false  | 打印ASM相关的调试信息                                        |
| save-result        | false  | 是否打开执行结果存日志功能，打开之后所有命令的运行结果都将保存到`/home/admin/logs/arthas/arthas.log`中 |
| job-timeout        | 1d     | 异步后台任务的默认超时时间，超过这个时间，任务自动停止；比如设置 1d, 2h, 3m, 25s，分别代表天、小时、分、秒 |

例如，想打开执行结果存日志功能，输入如下命令即可：

```
$ options save-result true                                                                           
```





## 四、Metrics-core

Metrics是一个记录监控指标的度量类型，类似与Logback记录日志一样，面向监控的度量数据。

```pom
  <dependencies>
        <!-- https://mvnrepository.com/artifact/io.dropwizard.metrics/metrics-core -->
        <dependency>
            <groupId>io.dropwizard.metrics</groupId>
            <artifactId>metrics-core</artifactId>
            <version>3.2.2</version>
        </dependency>
    </dependencies>
```

提供了五种基本度量组件：

- Gauge 度量值，包含简单度量值、比率信息。
- Counters 计数器
- Histograms 直方图
- Meters  qps计算器
- Timer 计时器

### 1、MetricRegistry

一个应用应只有一个MetricRegistry对象，存放此应用的所有Metric，每一个Metric都有一个唯一的名字。分为MetricRegistry和SharedMetricRegistries两个。SharedMetricRegistrics相比于MetricRegistry是线程安全的。



### 2、度量组件

#### （1）计数器Counter

记录次数

```java
MetricRegistry registry = new MetricRegistry();
// 1.定义counter的名字
Counter counter = registry.counter("query_gmv_count_sucess_count");

// 2.增加计数
// 默认值为1
counter.inc();
// 增加指定的数
counter.inc(3);

// 3.减少计数
// 3.1默认值是1
counter.dec();
// 3.2减少指定的数
counter.dec(2);

// 获取值
System.out.println(counter.getCount());
```



#### （2）计时器Timer

记录执行时间

```java
MetricRegistry registry = new MetricRegistry();
// 1.定义counter的名字
Timer timer = registry.timer("query_gmv_count_sucess_time");

// 2.打印时间
// 2.1开始计时
Timer.Context context = timer.time();
// 2.2暂停2秒,模拟执行任务
try {
	Thread.sleep(2000);
}catch (Exception e){

}
// 2.3 获取执行时间
System.out.println(context.stop()/1000000 + "mis");
```



#### （3）Gauge

一个接口，用来返回一个度量值，如下：

```java
public interface Gauge<T> extends Metric {
    /**
     * Returns the metric's current value.
     *
     * @return the metric's current value
     */
    T getValue();
}
```

​	它包括了

- 自定义Gauge，返回一个度量值
- RationGauge，返回一个比率值。
- Cached Gauges
- Derivative Gauges

1、自定义：

```java
public class RegistryGaugeTest {
    /**
     *自定义
     */
    public static void testGauge() {
        MetricRegistry registry = new MetricRegistry();
        Gauge gauge = registry.register("selfGuage", new Gauge<Integer>() {
            public Integer getValue() {
                return 1;
            }
        });

        System.out.println(gauge.getValue());
    }

    public static void main(String[] args){
        RegistryGuageTest.testGuage();
    }

}
```



2、RatioGauge

 一个抽象类，它的作用就是通过getValue（）来返回一个用来返回比值。代码如下： 

```java
public abstract class RatioGauge implements Gauge<Double> {
    /**
     * 定义Ration类，，包含两个成员变量，通过两个变量做除法，得到一个比率信息。
     */
    public static class Ratio {
        public static Ratio of(double numerator, double denominator) {
            return new Ratio(numerator, denominator);
        }

        private final double numerator;
        private final double denominator;

        private Ratio(double numerator, double denominator) {
            this.numerator = numerator;
            this.denominator = denominator;
        }

        /**
         * 返回Ration的比率信息
         *
         * @return the ratio
         */
        public double getValue() {
            final double d = denominator;
            if (isNaN(d) || isInfinite(d) || d == 0) {
                return Double.NaN;
            }
            return numerator / d;
        }

        @Override
        public String toString() {
            return numerator + ":" + denominator;
        }
    }

    /**
     * Returns the {@link Ratio} which is the gauge's current value.
     *
     * @return the {@link Ratio} which is the gauge's current value
     */
    protected abstract Ratio getRatio();

    @Override
    public Double getValue() {
        return getRatio().getValue();
    }
}
```



#### （4）Meter

统计事件发生率，如统计QPS，每一秒的请求次数。监控的数据有有：

- 总次数
- 平均吞吐量（总次数/总时间）
- 基于一分钟、五分钟和十分钟的指数加权移动平均

```java
public interface Metered extends Metric, Counting {
    /**
     * Returns 
     */
    long getCount();
    /**
     * 平均值 
     */
    double getMeanRate();
    /**
     * 15分钟
     */
    double getFifteenMinuteRate();
    /**
     * 5分钟
     */
    double getFiveMinuteRate();
    /**
     * 1分钟
     */
    double getOneMinuteRate();
}
```

例如：

```java
MetricRegistry registry = new MetricRegistry();
Meter meter = registry.meter("request_rate");
for (int i = 1; i < 5; i++) {
    try {
        Thread.sleep(i * 1000);
        meter.mark();
    } catch (Exception e) {
 
    }
}
// 返回总次数
System.out.println(meter.getCount());
// 返回平均速率=总次数/服务运行总时间。即每一秒中有多少次。
System.out.println(meter.getMeanRate());
// 返回基于15分钟指数加权移动平均
System.out.println(meter.getFifteenMinuteRate());
// 返回基于5分钟指数加权移动平均
System.out.println(meter.getFiveMinuteRate());
// 返回基于1分钟指数加权移动平均
System.out.println(meter.getOneMinuteRate());
```











































































































































































