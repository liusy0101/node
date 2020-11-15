# Docker

![img](https://img2018.cnblogs.com/blog/70544/201811/70544-20181128094617391-1947396056.png) 







## 一、是什么

Docker是一个开源的应用容器引擎，可以轻松的为任何应用创建一个轻量级的、可移植的、自给自足的容器。开发者在本地编译通过的容器可以批量的在生产环境上部署。

Docker类似于集装箱，各式各样的货物，经过集装箱的标准化进行托管，而集装箱与集装箱之前没有影响。Docker是一个开放平台，使开发人员和管理员可以在称为容器的松散隔离的环境中构建镜像、交互和运行分布式应用程序，以便在开发、QA和生产环境之间进行高效的应用程序生命周期管理。

 ![img](https://img2018.cnblogs.com/blog/70544/201811/70544-20181128093811704-1643592196.png) 



## 二、基本概念

### 一、镜像

一个特殊的文件系统。操作系统分为内核和用户空间，对于Linux来说，内核启动后会挂载root文件系统为其提供用户控件的支持。而Docker镜像，就相当于是一个root文件系统。

Docker镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含一些为运行时准备的配置参数。镜像不包含任何动态数据，其内容在构建之后也不会被改变。

Docker设计时，就充分利用了UnionFS的技术，将其设计为分层存储的架构，镜像实际是由多层文件系统联合组成。镜像构建时，会一层一层构建，前一层是后一层的基础。每一层构建完就不会再改变，后一层上的任何改变只发生在当前层。比如：删除前一层文件的操作，实际不是真的删除前一层的文件，而是仅把当前层标记为该文件已删除。

在最终容器运行的时候，虽然不会看到这个文件，但是实际上该文件会一直跟随镜像。因此，在构建镜像的时候，需要额外小心，每一层尽量只包含该层需要添加的东西，任何额外的东西应该在该层构建结束前清理掉。

分层存储的特征还使得镜像的复用、定制变的更为容易。甚至可以用之前构建好的镜像作为基础层，然后进一步添加新的层，以定制自己所需的内容，构建新的镜像。



### 二、容器（镜像运行的实体）

镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、暂停、停止、删除等。

容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行与属于自己独立的命名空间，容器也是分层存储。

容器存储层的生命周期跟容器一样，容器消亡时，容器存储层也会消亡，任何保存于容器存储层的信息都会丢失。、

按照Docker的要求，容器不应该向其存储层内写入任何数据，容器存储层也要保持无状态化。所有的文件写入操作，都应该使用数据卷、或者绑定宿主目录，在这些位置的读写会跳过存储层，直接对宿主发生读写，其性能和稳定性更高。容器消亡后数据卷的数据不会丢失。

 容器在整个应用程序生命周期工作流中提供以下优点：隔离性、可移植性、灵活性、可伸缩性和可控性。 最重要的优点是可在开发和运营之间提供隔离。 

 ![img](https://img2018.cnblogs.com/blog/70544/201811/70544-20181128094117580-274999612.png) 



### 三、仓库（集中存放镜像文件）

Docker Registry是一个集中存储、分发镜像的服务

一个Registry可以包含多个仓库（Repository），每个仓库可以包含多个标签（tag，也就是同一软件不同版本），每个标签对应一个镜像



## 三、Docker主要应用场景

### 一、简化配置

容器镜像一次打包，到处运行，无需针对各个平台独立配置。

### 二、流水线管理

提供一个从开发到上线均一致的环境，让代码的流水线变得简单不少。

### 三、提升开发效率

### 四、隔离应用

### 五、整合服务器

### 六、调试能力

### 七、多租户环境

### 八、快速部署



## 四、Docker持续开发工作流

![1596422268192](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596422268192.png)





## 五、Dockerfile

虽然可以通过docker commit命令来手动创建镜像，但是通过Dockerfile文件，可以帮助我们自动创建镜像，并且能够自定义创建过程。本质上，Dockerfile就是一系列命令和参数构成的脚本，这些命令应用于基础镜像并最终创建一个新的镜像，简化了从头到尾的构建流程并极大地简化了部署工作。

使用Dockerfile的优点：

- 像编程一样构建镜像，支持分层构建及缓存。
- 可以快速而精确的重新创建镜像以便于维护和升级。
- 便于持续集成。
- 可在任何地方快速构建镜像。



### 一、**Dockerfile指令：**

#### 1、FORM

用于设置在新镜像创建过程期间使用的容器镜像

格式：FROM 镜像名称

例如：FORM nginx

#### 2、RUN

指定要运行并捕获到新容器镜像中的命令，包括安装软件、创建文件等

格式：RUN ["","",""]

例如：RUN apt-get update

​			RUN mkdir -p /user/soft/redis

​			RUN apt-get update && mkdir -p /user/soft/redis

每一个指令都会创建一层，并构成新的镜像。如果指令够多的话，会变得很臃肿，增加构建时间，很容易出错，所以可以将指令合并执行，例如RUN apt-get update && mkdir -p /user/soft/redis

#### 3、COPY

将文件或目录复制到容器的文件系统中，需相对于Dockerfile的路径。

格式：COPY  文件路径

 如果源或目标包含空格，请将路径括在方括号和双引号中 

COPY ["",""]

#### 4、ADD

与COPY类似，还可以使用URL规范从远程位置复制

格式：ADD  <source>  <distination>

例如：ADD  https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe 

#### 5、WORKDIR

 用于为其他 Dockerfile 指令（如 RUN、CMD）设置一个工作目录，并且还设置用于运行容器映像实例的工作目录。 \

格式：WORKDIR 路径

例如： WORKDIR  /usr/soft/ngxin

#### 6、CMD

 用于设置部署容器映像的实例时要运行的默认命令。 如果设置了多个，只会执行最后一个。

格式：CMD 命令

例如：CMD  c:\\Apache24\\bin\\httpd.exe -w 

#### 7、ENTERPOING

​	 配置容器启动后执行的命令，并且不可被 docker run 提供的参数覆盖。每个 Dockerfile 中只能有一个ENTRYPOINT，当指定多个时，只有最后一个起效。 

格式：ENTERPOINT ["",""]

例如： ENTRYPOINT ["dotnet", "Magicodes.Admin.Web.Host.dll"] 

#### 8、ENV

设置环境变量，以kv键值对存在

格式：ENV  k1=v1 k2=v2

例如：ENV  JAVA_HOME=/usr/soft/java

#### 9、ARG

构建参数，作用于ENV相同，不同的是ARG的参数只在构建镜像的时候起作用，也就是docker build的时候。

格式：ARG k=v

#### 10、EXPOSE

用来指定端口，是容器内的应用可以通过端口与外界交互

格式：EXPOSE 端口

例如：EXPOSE 80

#### 11、转义字符

有些Dockerfile指令需要多行才能完成，此时需要一个换行转义符：“\”

#### 12、.dockerignore

.dockerignore文件用于忽略镜像构建是非必须的文件，可以是开发文档，日志等

![1596424069532](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596424069532.png)

#### 13、VOLUME

定义匿名数据卷。在启动的时候忘记挂载数据卷，会自动挂载到匿名卷。避免数据由于容器重启而丢失，避免容器不断变大。

格式：

```
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>
```

#### 14、USER

用于执行后续命令的用户和用户组

格式 ： USER 用户名[:用户组]

例如： USER root:root

#### 15、HEALTHCHECK

用于指定某个程序或者指令来监控 docker 容器服务的运行状态。

格式：

```shell
HEALTHCHECK [选项] CMD <命令>：设置检查容器健康状况的命令
HEALTHCHECK NONE：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

HEALTHCHECK [选项] CMD <命令> : 这边 CMD 后面跟随的命令使用，可以参考 CMD 的用法。
```

#### 16、ONBUILD

用于延迟构建命令的执行。简单的说，就是 Dockerfile 里用 ONBUILD 指定的命令，在本次构建镜像的过程中不会执行（假设镜像为 test-build）。当有新的 Dockerfile 使用了之前构建的镜像 FROM test-build ，这是执行新镜像的 Dockerfile 构建时候，会执行 test-build 的 Dockerfile 里的 ONBUILD 指定的命令。

格式：

```
ONBUILD <其它指令>
```

#### 17、创建自定义Docker镜像

创建了Dockerfile文件之后，可以使用docker build命令创建镜像。

例如：

```
docker build Dockerfile所在路径 -t 镜像名称
```





## 六、Docker使用

### 一、运行Docker应用程序

命令：

run：如果本地没有镜像，则默认去镜像仓库拉取，然后启动

```shell
docker run --name 容器名字 --rm -it -p [ip:]主机端口:容器端口 镜像名称 [命令]
```

--name：指定容器名字，缺省会随机分配

-i：交互式

-t：终端

--network：指定网络，连到同一个网络的容器可以互连

--dns：配置网络dns

-h HOSTNAME 或者 --hostname=HOSTNAME： 设定容器的主机名，它会被写到容器内的 /etc/hostname 和 /etc/hosts。

--dns=IP_ADDRESS： 添加 DNS 服务器到容器的 /etc/resolv.conf 中，让容器用这个服务器来解析所有不在 /etc/hosts 中的主机名。

--dns-search=DOMAIN： 设定容器的搜索域，当设定搜索域为 .example.com 时，在搜索一个名为 host 的主机时，DNS 不仅搜索 host，还会搜索 host.example.com。

-p：指定端口映射关系，也可加上ip

--rm：容器退出后，自动删除容器

-d：后台运行

-e：设置环境变量

 --env-file=[]：从指定文件读入环境变量； 

-u：设置命令运行的用户

 -m：设置容器使用内存最大值； 

 --net="bridge": 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；

--link=[]：添加链接到另一个容器；

--expose=[]： 开放一个端口或一组端口；

-w：运行的工作目录

-P：容器内部端口随机映射到主机的高端口

-a： 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项； 

例如： 启动一个容器并且用这个容器来做命令操作，其pid1进程是一个和用户交互的程序 

```shell
docker run -a stdin -a stdout -it ubuntu /bin/bash
```

-v：绑定一个数据卷



### 二、镜像使用

#### 1、查看本地已有镜像列表

```shell
docker images
```

#### 2、查找镜像

```shell
docker search 镜像名称：版本
```

#### 3、删除镜像

```shell
docker rmi 镜像名称：版本
```

#### 4、创建镜像

- 1、从已经创建的容器中更新镜像，并且提交这个镜像
- 2、使用 Dockerfile 指令来创建一个新的镜像

#### 5、更新镜像

进入容器，使用apt-get update进行更新，完成之后使用exit退出容器

提交容器副本：

```shell
docker commit -m="描述" -a="镜像作者" 容器id 要创建的目标镜像名
```

#### 6、设置镜像标签

```shell
docker tag 镜像id 镜像名称:tag
```



### 三、容器使用

#### 1、查看容器

```shell
#运行中的
docker ps

#所有运行过的
docker ps -a

#查看最后一次创建的容器
docker ps -l
```

#### 2、获取镜像

```shell
docker pull 镜像名称：版本
```

#### 3、启停已有的容器

```shell
docker start 容器id
docker stop 容器id
docker restart 容器id
```

#### 4、进入容器

```shell
#退出后，容器会停止
docker attach 容器id

#退出后，容器不会停止，推荐使用
#例如：docker exec -it 243c32535da7 /bin/bash
docker exec 容器id
```

#### 5、导入和导出容器

导出：

```shell
docker export 容器id > 导出文件名.tar
```

导入：

```shell
cat 导出文件名.tar | docker import 镜像名称：版本
```

#### 6、删除容器

```shell
docker rm -f 容器id
```

清除所有处于终止状态的容器

```shell
docker container prune
```

#### 7、查看容器映射端口

```shell
docker port 容器id
```

#### 8、查看容器日志

```shell
docker logs 容器id
```

#### 9、查看容器进程id

```shell
docker top 容器id
```

#### 10、查看容器信息

```shell
docker inspect 容器id
```

#### 11、新建docker网络

```shell
docker network create -d docker网络类型 网络名称
```

 **-d**：参数指定 Docker 网络类型，有 bridge、overlay。 

#### 12、配置DNS

可在主机上的/etc/docker/daemon.json文件增加如下内容来设置全部容器的DNS：

```
{
  "dns" : [
    "114.114.114.114",
    "8.8.8.8"
  ]
}
```

设置后，启动容器的 DNS 会自动配置为 114.114.114.114 和 8.8.8.8。配置完，需要重启 docker 才能生效

查看DNS是否生效：

```shell
docker run -it ubuntu  cat etc/resolv.conf
```

#### 13、杀掉正在运行的容器

```
docker kill -s KILL 容器id
```

 **-s :**向容器发送一个信号 

















## 七、部署或继续开发

部署之前，需要推送我们的镜像，使用docker push命令，可以将镜像推送到docker的官方镜像库，然后在通过docker pull命令拉取镜像进行部署。

```shell
docker push 镜像名称：镜像版本
```

推送之前，需要先登录到一个镜像仓库，如果未指定，默认是官方仓库Docker Hub，

```shell
docker login --username 用户名 --password 密码  [仓库地址]
```







## 八、Docker    Compose

简介：Compose是用于定义和运行多容器Docker应用程序的工具，通过Compose，可以使用YML文件来配置应用程序需要的所有服务，然后，使用一个命令，就可以从YML文件配置中创建并启动所有服务。

### 一、YAML

 YAML 是 "YAML Ain't a Markup Language"（YAML 不是一种标记语言）的递归缩写。 YAML 的语法和其他高级语言类似，并且可以简单表达清单、散列表，标量等数据形态。它使用空白符号缩进和大量依赖外观的特色，特别适合用来表达或编辑数据结构、各种配置文件、倾印调试内容、文件大纲（例如：许多电子邮件标题格式和YAML非常接近）。YAML 的配置文件后缀为 **.yml**，如：**runoob.yml** 。	



#### 1、基本语法

- 大小写敏感
- 使用缩进表示层级关系
- 缩进不允许使用tab，只允许空格
- 缩进的空格数不重要，只要相同层级的元素左对齐即可
- '#'表示注释



#### 2、数据类型

YAML 支持以下几种数据类型：

- 对象：键值对的集合，又称为映射（mapping）/ 哈希（hashes） / 字典（dictionary）
- 数组：一组按次序排列的值，又称为序列（sequence） / 列表（list）
- 纯量（scalars）：单个的、不可再分的值



#### 3、YAML 对象

对象键值对使用冒号结构表示 **key: value**，冒号后面要加一个空格。

也可以使用 **key:{key1: value1, key2: value2, ...}**。

还可以使用缩进表示层级关系；

```
key: 
    child-key: value
    child-key2: value2
```

较为复杂的对象格式，可以使用问号加一个空格代表一个复杂的 key，配合一个冒号加一个空格代表一个 value：

```
?  
    - complexkey1
    - complexkey2
:
    - complexvalue1
    - complexvalue2
```

意思即对象的属性是一个数组 [complexkey1,complexkey2]，对应的值也是一个数组 [complexvalue1,complexvalue2]



#### 4、YAML 数组

以 **-** 开头的行表示构成一个数组：

```
- A
- B
- C
```

YAML 支持多维数组，可以使用行内表示：

```
key: [value1, value2, ...]
```

数据结构的子成员是一个数组，则可以在该项下面缩进一个空格。

```yml
-
 - A
 - B
 - C
```



一个相对复杂的例子：

```shell
companies:
    -
        id: 1
        name: company1
        price: 200W
    -
        id: 2
        name: company2
        price: 500W
```

意思是 companies 属性是一个数组，每一个数组元素又是由 id、name、price 三个属性构成。

数组也可以使用流式(flow)的方式表示：

```shell
companies: [{id: 1,name: company1,price: 200W},{id: 2,name: company2,price: 500W}]
```



#### 5、复合结构

数组和对象可以构成复合结构，例：

```
languages:
  - Ruby
  - Perl
  - Python 
websites:
  YAML: yaml.org 
  Ruby: ruby-lang.org 
  Python: python.org 
  Perl: use.perl.org
```

转换为 json 为：

```
{ 
  languages: [ 'Ruby', 'Perl', 'Python'],
  websites: {
    YAML: 'yaml.org',
    Ruby: 'ruby-lang.org',
    Python: 'python.org',
    Perl: 'use.perl.org' 
  } 
}
```

#### 6、纯量

纯量是最基本的，不可再分的值，包括：

- 字符串
- 布尔值
- 整数
- 浮点数
- Null
- 时间
- 日期

使用一个例子来快速了解纯量的基本使用：

```
boolean: 
    - TRUE  #true,True都可以
    - FALSE  #false，False都可以
float:
    - 3.14
    - 6.8523015e+5  #可以使用科学计数法
int:
    - 123
    - 0b1010_0111_0100_1010_1110    #二进制表示
null:
    nodeName: 'node'
    parent: ~  #使用~表示null
string:
    - 哈哈
    - 'Hello world'  #可以使用双引号或者单引号包裹特殊字符
    - newline
      newline2    #字符串可以拆成多行，每一行会被转化成一个空格
date:
    - 2018-02-17    #日期必须使用ISO 8601格式，即yyyy-MM-dd
datetime: 
    -  2018-02-17T15:02:31+08:00    #时间使用ISO 8601格式，时间和日期之间使用T连接，最后使用+代表时区
```

#### 7、引用

**&** 锚点和 ***** 别名，可以用来引用:

```
defaults: &defaults
  adapter:  postgres
  host:     localhost

development:
  database: myapp_development
  <<: *defaults

test:
  database: myapp_test
  <<: *defaults
```

相当于:

```
defaults:
  adapter:  postgres
  host:     localhost

development:
  database: myapp_development
  adapter:  postgres
  host:     localhost

test:
  database: myapp_test
  adapter:  postgres
  host:     localhost
```

**&** 用来建立锚点（defaults），**<<** 表示合并到当前数据，***** 用来引用锚点。

下面是另一个例子:

```
- &showell Steve 
- Clark 
- Brian 
- Oren 
- *showell 
```

转为 JavaScript 代码如下:

```
[ 'Steve', 'Clark', 'Brian', 'Oren', 'Steve' ]
```





### 二、Compose使用的三个步骤

1、使用Dockerfile定义应用程序的环境

2、使用docker-compose.yml定义构成应用程序的服务，这样他们可以在隔离环境中一起运行。

3、执行docker-compose up命令来启动并运行整个应用程序。

docker-compose.yml案例：

```shell
# yaml 配置实例
version: '3'
services:
  web:
    build: .
    ports:
   - "5000:5000"
    volumes:
   - .:/code
    - logvolume01:/var/log
    links:
   - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```



### 三、Compose安装









