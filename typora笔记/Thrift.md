[toc]


# 跨语言RPC框架Thrift

Thrift是用来可伸缩的、跨语言的服务开发，通过一个代码生成引擎来构建高效、无缝的服务，这些服务能够实现跨语言调度

可通过homebrew直接安装

```shell
brew install thrift
```



## 1、Thrift支持的类型

### 1. 基本类型

bool：布尔值（true或者false）
byte：8位的有符号字节（java的byte类型）
i16：16位的有符号整数（java的short类型）
i32：32位的有符号整数（java的int类型）
i64：64位的有符号长整型（java的long类型）
double：一个64位的浮点数（java的double类型）
string： 一个utf8编码的字符串文本（java的String）

### 2. Structs

Thrift的structs用来定义一个通用对象，但是没有继承关系。

### 3.集合类型

list：一个有序的元素列表。元素可以重复。
set：一个无序的元素集合，集合中元素不能重复。
map：一个键值对的数据结构，相当于Java中的HashMap。

### 5.异常类型Exceptions

Thrift的异常类型，除了是继承于静态异常基类以外，其他的跟struct是类似的。表示的是一个异常对象。

### 6.服务类型Services

Thrift 的service类型相当于定义一个面向对象编程的一个接口。Thrift的编译器会根据这个接口定义来生成服务端和客户端的接口实现代码。





## 2、一个简单的Thrift调用实例

### 1. 编写.thrift文件，也就是IDL（接口描述语言）文件
以下是data.thrift文件：
```
namespace java thrift.generated
namespace py py.thrift.generated

typedef i16 short
typedef i32 int
typedef i64 long
typedef bool boolean
typedef string String

struct Person {
    1: optional String username,
    2: optional int age,
    3: optional boolean married
}

exception DataException {
    1: optional String message,
    2: optional String callStack,
    3: optional String date
}

service PersonService {
    Person getPersonByUsername(1: required String username) throws (1: DataException dataException),

    void savePerson(1: required Person person) throws (1: DataException dataException)
}
```

### 2. 使用thrift的编译器，生成客户端和服务端的代码
生成java的客户端服务端代码：

thrift --gen java src/thrift/data.thrift

生成Python的客户端服务端代码：

thrift --gen py src/thrift/data.thrift

其他语言的生成也是类似。

### 3. Thrift的调用
以java作为服务端，java作为客户端以及Python作为客户端对服务端进行请求。
java服务端代码：
```
public class PersonServiceImpl implements PersonService.Iface {

    @Override
    public Person getPersonByUsername(String username) throws DataException, TException {
        System.out.println("Got client param: " + username);
        Person person = new Person();
        person.setUsername(username);
        person.setAge(20);
        person.setMarried(false);

        return person;
    }

    @Override
    public void savePerson(Person person) throws DataException, TException {
        System.out.println("Got client param: ");

        System.out.println(person.getUsername());
        System.out.println(person.getAge());
        System.out.println(person.isMarried());
    }
}
```

```
public class ThriftServer {

    public static void main(String[] args) throws Exception {
        TNonblockingServerSocket serverSocket = new TNonblockingServerSocket(8899);
        THsHaServer.Args arg = new THsHaServer.Args(serverSocket).minWorkerThreads(2).maxWorkerThreads(4);

        PersonService.Processor<PersonServiceImpl> processor = new PersonService.Processor<>(new PersonServiceImpl());

        arg.protocolFactory(new TCompactProtocol.Factory());
        arg.transportFactory(new TFramedTransport.Factory());
        arg.processorFactory(new TProcessorFactory(processor));

        TServer server = new THsHaServer(arg);

        System.out.println("Thrift Server Started!");

        server.serve();
    }
}
```

java客户端代码：
```
public class ThriftClient {

    public static void main(String[] args) {
        TTransport transport = new TFramedTransport(new TSocket("localhost", 8899), 600);
        TProtocol protocol = new TCompactProtocol(transport);
        PersonService.Client client = new PersonService.Client(protocol);

        try {
            transport.open();

            Person person = client.getPersonByUsername("张三");
            System.out.println(person.getUsername());
            System.out.println(person.getAge());
            System.out.println(person.isMarried());

            System.out.println("------------");

            Person person1 = new Person();
            person1.setUsername("李四");
            person1.setAge(30);
            person1.setMarried(true);
            client.savePerson(person1);
        } catch (Exception e) {
            throw new RuntimeException(e.getMessage(), e);
        } finally {
            transport.close();
        }
    }

}
```

python客户端代码：

```
from py.thrift.generated import PersonService
from py.thrift.generated import ttypes

from thrift import Thrift
from thrift.transport import TSocket
from thrift.transport import TTransport
from thrift.protocol import TCompactProtocol

import sys

reload(sys)
sys.setdefaultencoding('utf8')

try:
    tSocket = TSocket.TSocket('localhost', 8899)
    tSocket.setTimeout(600)

    transport = TTransport.TFramedTransport(tSocket)
    protocol = TCompactProtocol.TCompactProtocol(transport)
    client = PersonService.Client(protocol)

    transport.open()

    person = client.getPersonByUsername('张三')

    print person.username
    print person.age
    print person.married

    print '------------------'

    newPerson = ttypes.Person()
    newPerson.username = '李四'
    newPerson.age = 30
    newPerson.married = True

    client.savePerson(newPerson)

except Thrift.TException, tx:
    print '%s' % tx.message
```

先运行服务端，然后再运行客户端，观察服务端和客户端的输出来理解下thrift的一个调用流程是什么样的。



## 3、Thrift的架构原理
以下是thrift的客户端和服务端交互的一个原理图：



![这里写图片描述](typora-user-images/SouthEast.png)


如上图，客户端在进行远程方法调用时，首先是通过Thrift的编译器生成的客户端，将调用信息（方法名，参数信息）以指定的协议进行封装，而
传输层TTransport是对协议层的封装进行处理（比如封装成帧frame），并通过网络发送出去。服务端这边流程跟客户端相反，收到客户端发过来
的数据后，首先经过传输层对传过来的数据进行处理，然后使用特定的协议（跟客户端是一一对应的）进行解析，然后再通过生成的Processor调用
用户编写的代码，如果有返回值的话，返回值以逆向的顺序，即通过协议层封装，然后传输层处理对数据进行发送，到了客户端那边就是对服务端返
回的数据进行处理，使用特定协议进行解析，然后得到一个调用个的结果。

以上就是Thrift的RPC调用的一个完整流程。

## 4、 Thrift的传输格式（协议层）
Thrift之所以被称为一种高效的RPC框架，其中一个重要的原因就是它提供了高效的数据传输。
以下是Thrift的传输格式种类：

- TBinaryProtocol: 二进制格式。效率显然高于文本格式。
- TCompactProtocol：压缩格式。在二进制基础上进一步压缩。
- TJSONProtocol：JSON格式。
- TSimpleJSONProtocol：提供JSON只写协议（缺少元数据信息），生成的文件很容易用过脚本语言解析。
- TDebugProtocol：使用易懂的刻度文本格式，以便于调试。
以上可以看到，在线上环境，使用TCompactProtocol格式效率是最高的，同等数据传输占用网络带宽是最少的。

## 5、Thrift的数据传输方式（传输层）

- TSocket：阻塞式socket。
- TFramedTransport：以frame为单位进行传输，非阻塞式服务中使用。
- TFileTransport：以文件形式进行传输。
- TMemoryTransport：将内存用于I/O，Java是现实内部实际使用了简单的ByteArrayOutputStream。
- TZlibTransport：使用zlib进行压缩，与其他传输方式联合使用。当前无java实现。



## 6、Thrift的服务模型

(1) TSimpleServer

简单的单线程服务模型，常用于测试。只在一个单独的线程中以阻塞I/O的方式来提供服务。所以它只能服务一个客户端连接，其他所有客户端在被服务器端接受之前都只能等待。

(2) TNonblockingServer：单线程

它使用了非阻塞式I/O，使用了java.nio.channels.Selector，通过调用select()，它使得程序阻塞在多个连接上，而不是单一的一个连接上。TNonblockingServer处理这些连接的时候，要么接受它，要么从它那读数据，要么把数据写到它那里，然后再次调用select()来等待下一个准备好的可用的连接。通用这种方式，server可同时服务多个客户端，而不会出现一个客户端把其他客户端全部“饿死”的情况。缺点是所有消息是被调用select()方法的同一个线程处理的，服务端同一时间只会处理一个消息，并没有实现并行处理。

(3)THsHaServer（半同步半异步server）：主从模型

针对TNonblockingServer存在的问题，THsHaServer应运而生。它使用一个单独的线程专门负责I/O，同样使用java.nio.channels.Selector，通过调用select()。然后再利用一个独立的worker线程池来处理消息。只要有空闲的worker线程，消息就会被立即处理，因此多条消息能被并行处理。效率进一步得到了提高。

(4)TThreadedSelectorServer：主从多线程模型

它与THsHaServer的主要区别在于，TThreadedSelectorServer允许你用多个线程来处理网络I/O。它维护了两个线程池，一个用来处理网络I/O，另一个用来进行请求的处理。

(5)TThreadPoolServer

它使用的是一种多线程服务模型，使用标准的阻塞式I/O。它会使用一个单独的线程来接收连接。一旦接受了一个连接，它就会被放入ThreadPoolExecutor中的一个worker线程里处理。worker线程被绑定到特定的客户端连接上，直到它关闭。一旦连接关闭，该worker线程就又回到了线程池中。
这意味着，如果有1万个并发的客户端连接，你就需要运行1万个线程。所以它对系统资源的消耗不像其他类型的server一样那么“友好”。此外，如果客户端数量超过了线程池中的最大线程数，在有一个worker线程可用之前，请求将被一直阻塞在那里。
如果提前知道了将要连接到服务器上的客户端数量，并且不介意运行大量线程的话，TThreadPoolServer可能是个很好的选择。

## 7、 其他

Facebook开源了一个简化thrift java开发的一个库——swift。swift是一个易于使用的、基于注解的java库，主要用来创建thrift可序列化类型和服务。

github地址：https://github.com/facebook/swift

十、 总结
Thrift是一个跨语言的RPC框架，如果有跨语言交互的业务场景，Thrift可能是一个很好的选择。如果使用恰当，thrift将是一个非常高效的一个RPC框架。开发时应根据具体场景选择合适的协议，传输方式以及服务模型。缺点就是Thrift并没有像dubbo那样提供分布式服务的支持，如果要支持分布式，需要开发者自己去开发集成。



























