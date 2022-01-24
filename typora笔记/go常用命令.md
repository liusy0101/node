[toc]



# GO 常用命令



![image-20220124185309446](typora-user-images/image-20220124185309446.png)



go.mod 和 go.sum是mod包管理工具产生的





# 1、go build



go build只会编译main包下的main文件

编译生成的二进制文件在当前目录





# 2、go install

 $GOPATH 这个环境变量，这个目录是工作目录，也就是说平时写代码时就用的这个目录，这个目录下一般有三个子目录

```
.
├── bin
├── pkg
├── src
```

分别是 binary，package，source 的缩写，我们容易理解它们的意思。

binary 即二进制，也就是说可执行文件，一般用于放系统使用的工具，如使用 go 命令装了一个命令行工具 tldr，那么可执行文件就放在这个目录，然后在系统上配置好环境变量，就可以直接使用这个工具。既然是可执行文件，那么编写源码的时候就需要有 main 包 和 main 函数，所以这里放置编译 main 包 main 函数后的文件。

package 即包，也就是库文件，这里同样放置二进制可行性文件，但是不同于 bin 目录下放置工具，这里放置一些公共文件，也就是供其他文件使用的库文件。所以编写源码时没有 main 包 main 函数，如我们常用的 github 下许多库文件就放在这个目录。

source 即源码，所以这个目录用于放置我们实际的项目文件。







go install 用于编译和安装包，安装就是把可行性文件放置到特定目录再配置环境变量。



go install编译后的文件在特定目录，没有入口函数（main）的移动到pkg目录，有入口函数的移动到bin目录中









# 3、go get

代码管理工具，拉取、编译、安装远程包

mod 开启时，go get 的下载路径为 $GOPATH/pkg/mod, 当 mod 关闭时，go get 的下载路径为 $GOPATH/src。







# 4、go run

编译并运行main包



使用 `go run main.go` 或 `go run /gee/main.go` 会得到结果 `Hello World`

- 如果要查看过程，则可以加上 v 参数

```
go run -v main.go
```

- 如果要查看临时目录的路径，则加上 work 参数

```
go run -work main.go
```

会得到

```
WORK=/tmp/go-build409560032
Hello World
```

我们会发现临时目录是 /tmp/go-buildxxx

- 如果需要使用命令行参数，那么直接在 package 后加上即可，即 arguments 参数

```
package main

import (
	"fmt"
	"os"
)

func main() {
	fmt.Println(os.Args)
}
```

查看 os.Args 源码

```
var Args []string
```

发现 Args 是个 string 切片，第一个元素是文件的路径

```
go run main.go hhh
```

会得到

```
[/tmp/go-build246132840/b001/exe/main hhh]
```







# 5、go mod

mod机制把所有依赖都放在 $GOPATH/pkg/mod目录下，里面可存放不同版本的依赖，然后在每个项目中都有一个mod文件，



![image-20220124195735115](typora-user-images/image-20220124195735115.png)



```
go mod init
```

命令初始化建立 go.mod 文件，现在就开启 mod 支持了。





go mod tidy 命令会删除那些没有 import，但是 require 了的包。然后会把 import 了包放到 require 中，然后到 $GOPATH/pkg/mod 中检查这个包是否存在，如果存在，则导入这个包，如果不存在，则拉取这个包。所以这个命令可以用来清楚没有使用的依赖，导入依赖和批量拉取依赖。

- 如果 go mod 中有些依赖我们没有使用，可以使用

```
go mod tidy
```

命令清除没有使用的依赖。

- 如果我们使用了 go mod ，明明本地也有这个库，但是却无法使用，如在 goland 中 报红，我们同样可以使用

```
go mod tidy
```

命令，这个命令会去 $GOPATH/pkg/mod 路径查找。因为虽然使用了 go mod ，但是 go mod 并没有扫描 /pkg/mod 路径，不能获取本地库列表，如果使用 Goland 的话，新建 go mod 文件时，就有一个 Index 弹窗用于扫描目录。

- 如果我们有些依赖本地没有，想要拉取，可以使用

```
go mod tidy
```

命令批量拉取。









```
go mod vendor
```

命令会在项目根目录新建 vendor 目录，会把依赖复制到这个目录。





如果使用末个包，但是这个包改名了，这时候想要继续使用原来的包名，又要拉取最新的包。或者使用末个包，但是这个包在原来的地方下载不了，只能到其他地方下载，那么可以使用 replace 关键字

如 goland.org/x/text 包在国内无法下载，但是 github 上有备份，那么可以使用

```
replace 
	golang.org/x/text v0.3.0 => github.com/golang/text v0.3.0
)
```

到 github 拉取会当做 goland.org/x/text 包使用。







# 6、go env

用于查看和设置go环境变量



```
go env [-json] [-u] [-w] [var ...]
```

默认没有参数会以行模式输出所有环境变量，可以输入 `go env` 查看，由又环境变量比较多，这里只介绍常用的几个。

如果要查看具体的某个或多个的环境变量，直接在 env 后加上即可，即 var 参数

如 `go env GOPATH GOROOT`，我的结果

```
/home/edte/go
/usr/lib/go
```

如果想要输出 json 格式的结果，则加上 json 参数

```
go env -json GOPATH GOROOT
{
        "GOPATH": "/home/edte/go",
        "GOROOT": "/usr/lib/go"
}
```

如果要删除末个变量，则可以直接设为空字符，或者使用 u 参数，即 unset

如 `go env -u GOBIN`

如果要更改末个变量，则需要使用 w 参数，即 write

如 `go env -w GOPROXY=https://goproxy.io,direct`





常见的环境变量

- GOROOT

GOROOT 是刚安装 go 时会遇到的一个环境变量， GOROOT 用于存放 go 的安装路径，如标准库，工具链等就放在这个位置。一般安装 go 时会默认设置，从而无需手动设置，win 下一般在 `C:\go`，可以使用 `go env GOROOT` 查看。

- GOPATH

GOPATH 也是安装 go 时会遇到的一个环境变量，用于存放工作目录。也就是实际工作时用到的目录，我们在 go install 中已经讲解了这个路径的具体作用。这个目录用于存放实际的项目源码以及用到的远程库文件。

- **GOBIN**

GOBIN 是默认的一个环境变量，一般不用设置。我们在 go install 中说过，安装可执行程序时，会安装到 $GOPATH/bin 目录，这个目录就是 GOBIN ，我们也可以更改，不过一般不建议改。

- GOOS 和 GOARCH

go 本身支持交叉编译，也就是可以在一个平台上生成另一个平台的可执行程序。因此需要设置目标操作系统和目标系统架构。GOOS 即 go os ，os 即操作系统，GOARCH 即 go arch，arch 即体系架构。

```
$ go env -json GOOS GOARCH  
{
        "GOARCH": "amd64",
        "GOOS": "linux"
}
```

可以看到这两个值默认是本机的。如我们要生成 64 位的 win，使用 `go env -w GOOS=windows` 设置 os，再使用 `go build main.go` 就得到了 `main.exe`,我们知道 exe 是 win 下可执行程序的后缀，故成功了。

如果目标系统是 mac 或 linux ，可以使用 `uanme -a` 查看，如果是 win ，直接看 32 位还是 64 位即可。





- GOPROXY

使用 go mod 管理包拉取依赖时，包服务器一般在国外，拉取速度比较慢，一般我们可以自己设置代理，不过 go mod 官方自带了一个 代理方式，通过修改 GOPROXY 变量即可修改 go mod 拉取的代理。

```
go env -w GOPROXY=https://goproxy.cn,direct
```

- GO111MODULE

GO111MODULE 即 go1.11 版本推出的 go mod 机制。go mod 是最新的一种包管理工具。在 1.12 前 go mod 是默认关闭的，不过 1.13 后便默认开启了

| 状态 |          功能           |
| :--: | :---------------------: |
|  on  |       打开 go mod       |
| off  |       关闭 go mod       |
| auto | 自动判断是否打开 go mod |

最新版一般都是默认打开的，建议不要修改。





























