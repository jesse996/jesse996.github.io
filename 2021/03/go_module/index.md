# Go_module


# GOPATH 的前世今生

GOPATH 是布置 Go 开发环境时所设置的一个环境变量。历史版本的 go 语言开发时，需要将代码放在 GOPATH 目录的 src 文件夹下。go get 命令获取依赖，也会自动下载到 GOPATH 的 src 下。

例如：

`go get github.com/foo/bar`
会将代码下载到 `$GOPATH/src/github.com/foo/bar`

可以通过命令 `go env` 获取当前设置的 GOPATH 路径。

例如我的电脑上设置如下：
`GOPATH="/Users/xxx/project/go"`

GOPATH 具体结构如下，必须包含三个文件夹，具体如下图所示：

```
GOPATH
├── bin //编译生成的二进制文件
├── pkg //预编译文件，以加快程序的后续编译速度
|── src //所有源代码
├── github.com
...
...
```

# GOPATH 模式不再推荐的原因

最严重的问题就是，GOPATH 模式下，go get 命令使用时，没有版本选择机制，拉下来的依赖代码都会默认当前最新版本，而且如果当项目 A 和项目 B 分别依赖项目 C 的两个不兼容版本时， GOPATH 路径下只有一个版本的 C 将无法同时满足 A 和 B 的依赖需求。这可以说是一个很大的缺陷了，因而 Go1.13 起，官方就不再推荐使用 GOPATH 的模式了。

GOPATH 模式衍生出的版本管理工具的进化
随着 Go 语言使用人数的增长，依赖包的丰富，依赖版本问题尤其严重。

于是 Go 官方在 Go 1.5 的时候提出了实验性质的 vendor 机制：每个项目都可以有一个 vendor/ 目录来存放项目所需版本依赖的拷贝。

社区中基于官方给的机制，开发出了各种版本管理工具。比较流行的比如 govendor，以及之前曾被官方认定的 godep 工具等。

这些工具的思路基本都是为每个项目单独维护一份对应版本依赖的拷贝。

管理工具虽然丰富了起来，但是不同版本工具之间不兼容，无法协作，各种工具还都有学习成本。这时候 在 Go 官方扶持下成立的 dep 项目被大家认为是未来一统江湖的版本管理工具，被称作 official experiment。

dep 采用了和 Rust 的管理工具 Cargo 类似的管理模式，原理在此不深究。

没过多久，Go 社区的核心人物 rsc 提出了 vgo 方案。一时间竟然出现了两个所谓的 Go 官方的版本管理方案。最终官方采用了 vgo 方案，随着 vgo 的逐渐成熟，Go 1.11 发布了该功能，并集成到了 Go 的官方工具中，也就是当前的 Go modules。

## Go modules 相关环境变量

使用 Go modules，一般涉及到如下 6 个环境变量的配置，go env 可以列出，摘录如下：

```
GO111MODULE="auto"
GOPROXY="https://goproxy.io,direct"
GONOPROXY=""
GOSUMDB="sum.golang.org"
GONOSUMDB=""
GOPRIVATE=""
```

`GO111MODULE`
是开启使用 Go modules 的开关，可用参数值如下：

| 值   | 含义                                                                                                     |
| ---- | -------------------------------------------------------------------------------------------------------- |
| auto | 在 GOPATH/src 之外，将自动使用 Go Modules 模式。否则还是用 GOPATH 模式。目前在最新的 Go1.14 中是默认值。 |
| on   | 启用 Go modules，将不使用 GOPATH，推荐设置，将会是未来版本中的默认值。                                   |
| off  | 或者不设置 Go 将使用 GOPATH 和沿用老的 vendor 机制，禁用 Go modules。不推荐设置。                        |

设置开启 on 的方法

```
go env -w GO111MODULE=on
```

也可在 .bash_profile 文件中直接环境变量方式设置，增加

```
export GO111MODULE=on
```

## GOPROXY

用于设置 Go 模块代理，其作用是使 Go 在拉取模块版本时直接通过镜像站点来快速拉取默认值是这个默认地址，国内无法访问，所以开启 Go Modules 必需设置镜像代理地址

列举国内常用的镜像代理地址如下：

地址 简介
https://goproxy.io 一个全球代理为 Go 模块而生
https://mirrors.aliyun.com/goproxy/ 阿里镜像代理
https://goproxy.cn 七牛云赞助支持的

GOPROXY 的值是一个以英文逗号 “,” 分割的 Go 模块代理列表，允许设置多个模块代理

不使用，可设为 “off”

默认值中有个 direct，用来告诉 Go 将直接从依赖的源地址进行下载操作（比如 GitHub 等），例如当值列表中上一个 Go 代理返回 404 或 410 时，Go 自动尝试列表中的下一个，遇见 “direct” 时回到源地址去抓取。

最终：设置代理方法一般如下：如使用七牛网代理

```
go env -w GOPROXY=https://goproxy.cn,direct
```

也可在 .bash_profile 文件中直接环境变量方式设置，增加

```
export GOPROXY=https://goproxy.cn,direct
```

## GOSUMDB

Go checksum database 的缩写，含义如其名字，用于在拉取模块版本时保证拉取到的模块版本数据未经过篡改，若发现不一致，也就是可能存在篡改，将会立即中止。

GOSUMDB 的默认值为：sum.golang.org，在国内也是无法访问的，但是 GOSUMDB 可以被 Go 模块代理所代理，即设置 GOPROXY 自然就解决了。

若设置为“off”，就禁止 Go 在后续操作中校验模块版本。

## GONOPROXY / GONOSUMDB / GOPRIVATE

如果当前项目依赖了私有模块，则配置会涉及这三个环境变量。例如公司的私有 git 仓库，又或是 github 中的私有库，都是属于私有模块，都是要进行设置的，否则会拉取失败。简单来说就是应对，GOPROXY 设置的代理或 GOSUMDB 设定的 Go checksum database 代理无法访问模块时的情形。

建议直接设置 GOPRIVATE，它的值将作为 GONOPROXY 和 GONOSUMDB 的默认值，所以建议的最佳设置是直接使用 GOPRIVATE。

它们的值都是一个以英文逗号 “,” 分割的模块路径前缀，也就是可以设置多个，例如：

```
go env -w GOPRIVATE="\*.my.com,github.com/eabc/def"
```

设置后，前缀为 \*.my.com 和 github.com/eabc/def 的模块都会被认为是私有模块。

将不经过 Go module proxy 和 Go checksum database，需要注意的是不包括 my.com 本身。

## Go modules 操作命令及相关文件解读

可以命令行执行 go help mod 打印出 go mod 相关命令

```
download download modules to local cache 常用，下载依赖包
edit edit go.mod from tools or scripts ide 编辑就行
graph print module requirement graph 查看使用而已
init initialize new module in current directory 常用
tidy add missing and remove unused modules 常用
vendor make vendored copy of dependencies 从 mod cache 中拷贝到项目的 vendor
verify verify dependencies have expected content
why explain why packages or modules are needed
```

接下来主要介绍 go mod init 命令，及其涉及到的必要文件

## go mod init 命令

初始化项目，使用方法例子如下：创建个空项目之后执行如下命令

```
go mod init gitee.com/biexiaoyansudian/my.cn
```

指定了模块导入路径为 gitee.com/biexiaoyansudian/my.cn

这个 init 指定的路径作用是：

- 作为模块的标识（identity）

- 作为模块的 import path，当其他项目引用这个模块下的 package 时都会以该 import path 作为共同的前缀，比如：
  import "gitee.com/biexiaoyansudian/my.cn/mypkg"

go mod init 执行完毕，就初始化了使用 Go modules 的项目，会多出来一个 go.mod 文件。它记录了当前项目的模块信息，每一行都以一个关键词开头。

### go.mod 文件

此时，go.mod 文件内容如下

```
module gitee.com/biexiaoyansudian/my.cn

go 1.14
```

Go modules 模式下，使用 go get 命令，相关信息可以自动记录到 go.mod 文件中

看如下示例：

在刚才初始化的项目中，下载 gorm 扩展包

go get -u github.com/jinzhu/gorm

默认下载最新版本，Go modules 模式下 go get 可以进行版本指定 @ 版本管理的 tag，例如

go get -u github.com/jinzhu/gorm@v1.0.0

其拉取的结果缓存在 GOPATH/pkg/mod 和 GOPATH/pkg/sumdb 目录下，而在 mod 目录下会以 github.com/foo/bar 的格式进行存放。

下载完成后，go.mod 文件内容自动变更如下

module gitee.com/biexiaoyansudian/my.cn

go 1.14

require github.com/jinzhu/gorm v1.9.12 // indirect

//手动注释 indirect 标识表示该模块为间接依赖，也就是在当前应用程序中的 import 语句中，并没有发现这个模块的明确引用，如果没引用，我们提前先拉下来这个包，就会出现该注释，比如直接使用 go get 拉代码包，而不是 go build 让命令自动根据 go.mod 拉代码包

下面介绍下 go.mod 文件中可以使用到的语法关键词以及含义：

module： 定义当前项目的模块路径，不再赘述

go： 标识当前模块的 Go 语言版本，目前来看还只是个标识作用。

require： 说明 Module 需要什么版本的依赖。

exclude： 用于从使用中排除一个特定的模块版本。在实际的项目中很少被使用，故很少会显式的排除某个包的某个版本，除非我们知道某个版本有严重 bug。比如指令 exclude github.com/google/uuid v1.1.0，表示不使用 v1.1.0 版本。

场景举例：

当前项目 gitee.com/biexiaoyansudian/my.cn

下载依赖 go get github.com/google/uuid@v1.0.0

my.cn 的 go.mod 文件如下

module gitee.com/biexiaoyansudian/my.cn

go 1.14

require github.com/google/uuid v1.0.0 // indirect

此时另创建项目 gitee.com/biexiaoyansudian/exclue

下载依赖 go get -uu github.com/google/uuid@v1.1.0

此时如果当前 my.cn 项目想要引入 exclue 项目，可以看到 exclue 项目用的是 uuid@v1.1.0 大于 my.cn 项目版本，则会自动的将 my.cn 项目的也升级为 uuid@v1.1.0

my.cn 的 go.mod 文件如下

module gitee.com/biexiaoyansudian/my.cn

go 1.14

require (
gitee.com/biexiaoyansudian/exclue v1.0.0 // indirect
github.com/google/uuid v1.1.0
)

而如果在下载 exclue 依赖之前，在 my.cn 项目的 go.mod 文件里加上

exclude github.com/google/uuid v1.1.0

此时，再去拉 exclue 项目，可以发现 uuid 直接跳过了 v1.1.0 版本升级到了 v1.1.1

my.cn 的 go.mod 文件如下

module gitee.com/biexiaoyansudian/my.cn

go 1.14

require (
gitee.com/biexiaoyansudian/exclue v1.0.0 // indirect
github.com/google/uuid v1.1.1
)

exclude github.com/google/uuid v1.1.0

replace： 替换 require 中声明的依赖，使用另外的依赖及其版本号。
首先介绍个命令，可以查看项目最终所使用的全部依赖

go list -m all

使用场景一：替换 require 的包
用例子来说明 replace 的使用场景一，一般也是最不会使用的场景。

当前项目的 go.mod 如下

module gitee.com/biexiaoyansudian/my.cn

go 1.14

require (
gitee.com/biexiaoyansudian/exclue v1.0.0 // indirect
github.com/google/uuid v1.1.1
)

exclude github.com/google/uuid v1.1.0

下执行命令 go list -m all

gitee.com/biexiaoyansudian/my.cn
gitee.com/biexiaoyansudian/exclue v1.0.0
github.com/google/uuid v1.1.1

我们在 go.mod 下增加这样一句配置

replace github.com/google/uuid v1.1.1 => github.com/google/uuid v1.1.0

再次执行命令，结果如下

gitee.com/biexiaoyansudian/my.cn
gitee.com/biexiaoyansudian/exclue v1.0.0
github.com/google/uuid v1.1.1 => github.com/google/uuid v1.1.0

发现最终生效的由 uuid v1.1.1 被替换成了 uuid v1.1.0

一般来说，这种场景使用的不多，和直接修改 require 作用相同

生效有前提条件：

一是只有在当前引用的模块有效

二是 replace 命令左侧的包名和版本，必须是 require 中包含的包名和版本

场景二：替换无法下载的包
由于网络问题，有些包无法下载，比如 golang.org 下的包，而这些包在 GitHub 都有镜像，此时就可以使用 GitHub 上的包来替换。

比如，项目中使用了 golang.org/x/text 包：

module github.com/renhongcai/gomodule

go 1.14

require (
github.com/google/uuid v1.1.1
golang.org/x/text v0.3.2
)

replace github.com/google/uuid v1.1.1 => github.com/google/uuid v1.1.0

项目编译时就会从 GitHub 下载包。我们源代码中 import 路径 golang.org/x/text/xxx 不需要改变

场景三：调试依赖包
我们调试依赖包，就可以使用 replace 来修改依赖，如下所示：

replace (
github.com/google/uuid v1.1.1 => ../uuid
)

使用本地的 uuid 来替换依赖包，此时，我们可以任意地修改 …/uuid 目录的内容来进行调试。

除了使用相对路径，还可以使用绝对路径，甚至还可以使用自已的 fork 仓库。

场景四：禁止被依赖
另一种使用 replace 的场景是你的 module 不希望被直接引用，比如开源软件 kubernetes，在它的 go.mod 中 require 部分有大量的 v0.0.0 依赖，比如：

module k8s.io/kubernetes

require (
...
k8s.io/api v0.0.0
k8s.io/apiextensions-apiserver v0.0.0
k8s.io/apimachinery v0.0.0
...
)

由于上面的依赖都不存在 v0.0.0 版本，所以其他项目直接依赖 k8s.io/kubernetes 时会因无法找到版本而无法使用。

因为 Kubernetes 不希望作为 module 被直接使用，其他项目可以使用 kubernetes 其他子组件。kubernetes 对外隐藏了依赖版本号，其真实的依赖通过 replace 指定：

replace (
k8s.io/api => ./staging/src/k8s.io/api
k8s.io/apiextensions-apiserver => ./staging/src/k8s.io/apiextensions-apiserver
k8s.io/apimachinery => ./staging/src/k8s.io/apimachinery
...
)

go.sum 文件
我们发现，在 go.mod 下还会生成个 go.sum 文件

用途
下载依赖包有可能被恶意篡改，以及缓存在本地的依赖包也有被篡改的可能，单单一个 go.mod 文件并不能保证一致性构建，Go 开发团队在引入 go.mod 的同时也引入了 go.sum 文件，用于记录每个依赖包的哈希值（SHA-256 算法），在 build 时，如果本地的依赖包 hash 值与 go.sum 文件中记录得不一致，则会拒绝 build。

示例：格式[/go.mod]

github.com/google/uuid v1.0.0 h1:b4Gk+7WdP/d3HZH8EJsZpvV7EtDOgaZLtnaNGIu1adA=
github.com/google/uuid v1.0.0/go.mod h1:TIyPZe4MgqvfeYDBFedMoGGpEw/LqOeaOT+nhxU+yHo=

正常情况下，每个依赖包版本会包含两条记录，第一条记录为该依赖包版本整体（所有文件）的哈希值，第二条记录仅表示该依赖包版本中 go.mod 文件的哈希值，如果该依赖包版本没有 go.mod 文件，则只有第一条记录

go.sum 生成过程
在项目的根目录中执行 go get 命令的话，go get 会同步更新 go.mod 和 go.sum 文件，go.mod 中记录的是依赖名及其版本，如：

go.sum 文件中则会记录依赖包的哈希值（同时还有依赖包中 go.mod 的哈希值）

在更新 go.sum 之前，为了确保下载的依赖包是真实可靠的，go 命令在下载完依赖包后还会查询 GOSUMDB 环境变量所指示的服务器，以得到一个权威的依赖包版本哈希值。如果 go 命令计算出的依赖包版本哈希值与 GOSUMDB 服务器给出的哈希值不一致，go 命令将拒绝向下执行，也不会更新 go.sum 文件。

go mod tidy
拉取缺少的模块，移除不用的模块

go mod vendor
将 GOPATH/src/pkg/mod 中的缓存包，复制到项目的 vendor 目录中，即使用每个项目使用自身包的模式，类似之前的 govendor 管理方式

尽管如此，默认情况下，go build 将忽略该目录的内容。如果想从 vendor/ 目录中建立依赖关系，需要如下参数 build

go build -mod vendor

go mod download
下载依赖包

go mod edit
编辑 go.mod 文件一般直接用 ide 编辑就行

go mod graph
打印模块依赖图

go mod verify
验证依赖是否正确

go mod why
解释为什么需要依赖

补充与说明
关于自己发布包
go get 命令使用的版本是 git 中的 tag，我们制作包的时候也应该创建合适的 tag，例如

git tag v1.0.0
git push --tags

此时一个最佳实践是同时创建新分支 v1，以便后面我们修复错误和更新：

git checkout -b v1
git push -u origin v1

现在，我们可以使用 master 而不必担心破坏我们的版本。

接下来在 v1 分支中做修改，并将其标记为新版本。

git commit -m "修改"
git tag v1.0.1
git push --tags origin v1

始终要保持在 v1 分支中修改，这个分支是始终的进行兼容更新的，如果有大的不兼容的改版，要新建并切换分支 v2，发布新的路径。

模块的使用与更新
我们可以在写代码的时候先写好导入的包等，之后初始化 go mod 模式，go build 或者 go mod tidy 等命令，会自动下载相关导入

小版本更新
go get -u 会更新最新的小版本，举个例子，它会将 1.0.0 更新为 1.0.1 或者 1.1.0 这样类似的版本。go get -u=patch 来获取最新的 patch 更新。 举个例子，它会更新到 1.0.1 而不是 1.1.0 版本。

例子总结：

由于我们的程序使用的是软件包的 1.0.0 版，并且我们刚刚创建了 1.0.1 版，以下任何 命令都会将我们更新到 1.0.1 版:

go get -u
go get -u=patch
go get github.com/robteix/testmod@v1.0.1

大版本更新
根据语义版本语义，大版本不同于小版本。大版本会破坏向后兼容性。从 Go 模块的角度来看，大版本是一个完全不同的包。一个库的两个不兼容的版本，实质上是两个不同的库。其余的和以前一样，我们推它，把它标记为 v2.0.0 (并可选地创建一个 v2 分支。)

