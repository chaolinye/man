# Go 包管理

## GOPATH

go 1.2 之前使用环境变量 `GOPATH` 来指定 import 的路径

!> 使用 go modules 时，GOPATH 基本被废弃了，目前还用来指定modules 依赖包的存储路径(`GOPATH[0]/pkg/mod`)，但后续版本连这点作用也会被 `GOMODCACHE` 替代，这就真的没用了

!> go 1.2 之后不需要特别关注 GOPATH 了

### 基本结构

`GOPATH` 的值是一个或者多个路径，路径之间以分隔符(windows `;`, linux `:`) 隔开

`GOPATH` 下有三个子目录:

- `src`: import 寻找依赖的路径

- `pkg`: 依赖编译后的路径

- `bin`: go install 生成的可执行程序放置的目录，可设置环境变量 `GOBIN` 修改

比如 `GOPATH=/home/user/go`, 则其下的结构如下

```
    /home/user/go/
        src/
            foo/
                bar/               (go code in package bar)
                    x.go
                quux/              (go code in package main)
                    y.go
        bin/
            quux                   (installed command)
        pkg/
            linux_amd64/
                foo/
                    bar.a          (installed package object)
```

### 特殊子目录

- `internal`

- `vendor`

这两个子目录下的 package 只能由同一个父目录下的 package 引用，比如

```
    /home/user/go/
        src/
            crash/
                bang/              (go code in package bang)
                    b.go
            foo/                   (go code in package foo)
                f.go
                bar/               (go code in package bar)
                    x.go
                internal/
                    baz/           (go code in package baz)
                        z.go
                vendor/
                    crash/
                        bang/      (go code in package bang)
                            b.go
                quux/              (go code in package main)
                    y.go
```

两个目录的区别在于其下的包的引用方式的不同，`internal` 目录的引用方式和一般目录一致，`vendor` 目录只需要其下的一段, 比如

- `y.go` 中调用 `internal` 目录下的包: `import "foo/internal/baz"`

- `y.go` 中调用 `vendor` 目录下的包: `import "crash/bang"`


## Import 

### Import 的寻找过程

`GOPATH` 模式下，Go 中 import 的寻找过程

1. GOROOT: 标准库路径，一般是 Go 的安装目录

2. vendor: 特殊子目录，从当前目录及其上级目录找 vendor 目录

3. GOPATH: 可指定多个路径

4. 远程下载

`module-aware` 模式下，Go 中 import 的寻找过程

1. GOROOT: 标准库路径，一般是 Go 的安装目录

2. vendor: 特殊子目录，从当前目录及其上级目录找 vendor 目录

3. GOPATH[0]/pkg/mod: module 存储目录

4. 远程下载

### 远程下载

> Go 没有中央仓库，一般是直接从 VCS 仓库中获取

> [godoc.org](https://godoc.org/), [golang.org/pkg](https://golang.org/pkg/) 这两个网站有提供标准库和社区库的查询功能，以便于查找已有的库

Go 能够识别一些著名的 VCS 仓库， Import 这些仓库中的 package 可以直接使用它们的路径，Go 会自动识别路径对应的 VCS 类型，采用相应的下载方式

```
	Bitbucket (Git, Mercurial)

		import "bitbucket.org/user/project"
		import "bitbucket.org/user/project/sub/directory"

	GitHub (Git)

		import "github.com/user/project"
		import "github.com/user/project/sub/directory"

	Launchpad (Bazaar)

		import "launchpad.net/project"
		import "launchpad.net/project/series"
		import "launchpad.net/project/series/sub/directory"

		import "launchpad.net/~user/project/branch"
		import "launchpad.net/~user/project/branch/sub/directory"

	IBM DevOps Services (Git)

		import "hub.jazz.net/git/user/project"
		import "hub.jazz.net/git/user/project/sub/directory"
```

而其他域名的 VCS 仓库（比如公司自建的 Gitlab 仓库），则需要注明 VCS 的类型，Go 才能正确下载

相关源码: `$GOROOT/src/cmd/go/get/vcs.go`

```go
    // General syntax for any server.
	// Must be last.
	{
		regexp:         lazyregexp.New(`(?P<root>(?P<repo>([a-z0-9.\-]+\.)+[a-z0-9.\-]+(:[0-9]+)?(/~?[A-Za-z0-9_.\-]+)+?)\.(?P<vcs>bzr|fossil|git|hg|svn))(/~?[A-Za-z0-9_.\-]+)*$`),
		schemelessRepo: true,
	},
```

> 通过正则表达式从链接中提取仓库类型

不同 VCS 的类型后缀如下

```
	Bazaar      .bzr
	Fossil      .fossil
	Git         .git
	Mercurial   .hg
	Subversion  .svn
```

比如 package 放置在公司内建的 Gitlab 仓库中，路径是 `https://gitlab.mycompany.com/yechaolin/hello-go/hello`

则 import 的写法应该是 `import "gitlab.mycompany.com/yechaolin/hello-go.git/hello"`, 添加了 `.git` 注明类型

如果 import 的路径既不是既定的 VCS 仓库，路径中也没有注明 VCS 类型，Go 则会通过 https 或者 http 访问路径，并从路径的网页 meta 标签中获取源码的地址, 比如

```html
<head>
    <meta name="go-import" content="example.org git https://code.org/r/p/exproj">
</head>
```

通过 meta 获取到模块名是 `example.org`, 源码地址是 `https://code.org/r/p/exproj`, VCS 类型是 `Git`

> 用 meta 方式，就不需要源码存储地址和模块名一致了

> TODO 是不是可以通过 meta 的方式优化团队内部的模块引用

### FAQ

#### 不想在 import 的路径中嵌入仓库类型？

在 go modules 中， 如果不想在 import 的路径中嵌入仓库类型，可以使用 replace 进行替换, 如下

import 写法: `import "gitlab.mycompany.com/yechaolin/hello-go/hello"`

`go.mod` 中进行 replace

```
module demo
go 1.15

replace gitlab.mycompany.com/y00450147/hello-go => gitlab.mycompany.com/y00450147/hello-go.git v1.0.0

require gitlab.mycompany.com/y00450147/hello-go v1.0.0
```

#### VCS 仓库采用自定义端口，import 报错？

go module path 的有效字符:

`$GOROOT/src/cmd/vendor/golang.org/x/mod/module/module.go`

```go
func pathOK(r rune) bool {
	if r < utf8.RuneSelf {
		return r == '+' || r == '-' || r == '.' || r == '_' || r == '~' ||
			'0' <= r && r <= '9' ||
			'A' <= r && r <= 'Z' ||
			'a' <= r && r <= 'z'
	}
	return false
}
```

可见 go module path 不能含有 `:` 符号，即 require 和 replace 中都不能含有 `:` 字符，如果自建的 Git 仓库采用的是自定义的端口号(比如 `2222`), 可以通过 git 替换

```bash
git config --global url."git@gitlab.mycompany.com:2222".insteadOf "git@gitlab.mycompany.com"
```

> 更统一的做法是建个代理转发

## go modules

### 两种依赖模式

Go 官方目前有两种依赖模式

-  `GOPATH-mode`

- `module-aware mode`

> `module-aware mode` 在 `GO 1.12` 引入，之前的版本只有 `GOPATH-mode`

两种模式的选择根据环境变量 `GO111MODULE` 的值和 `go.mod` 文件的存在来决定

1. `GO111MODULE=on`, 则是 `module-aware mode`

2. `GO111MODULE=off`, 则是 `GOPATH-mode`

3. `GO111MODULE=auto` 或者没设置值， 如果当前目录或者上级目录含有 `go.mod` 文件，则是 `module-aware mode`, 否则是 `GOPATH-mode`

> 默认情况下 `GO111MODULE` 是没有设置值的，即根据 `go.mod` 文件的存在与否判断模式

> go 1.15 及之后版本强烈建议把 GO111MODULE 设置为 on，设置命令 `go env -w GO111MODULE=on`

GOPATH 模式下， import 的依赖是通过 `GOPATH` 环境变量定义的一个或多个路径来寻找的

module-aware 模式下，`GOPATH` 不再是 import 的搜索区域

module-aware 模式下， GOPATH 还剩两个作用

- 下载的依赖放置在 `GOPATH` 的第一个路径的 `$GOPATH[0]/pkg/mod` 文件夹下，后续应该会被 `GOMODCACHE` 替代

- `git install` 生成的可执行程序放在 `$GOPATH/bin` 下，后续应该会被 `GOBIN` 完全替代

> modules aware 模式的依赖管理方式和 java 的 maven 工具很像，都是在某个公共目录下存储依赖，不过没有提供中央仓库，而是通过分散地根据 URL 获取

### 创建一个 module-aware 工程

任意目录下，运行

```bash
go mod init demo
```

会在当前目录生成 `go.mod` 文件

```
# 模块名
module demo

go 1.15
```

编写 `main.go`

```
package main

import (
    "fmt"
)

func main() {
    fmt.Printlnn("hello world")
}
```

编译，运行

```bash
go runn main.go
```

### go.mod

`go.mod` 是启用了 Go moduels 的项目所必须的最重要的文件，它描述了当前项目（也就是当前模块）的元信息，每一行都以一个动词开头，目前有以下 5 个动词:

- module：用于定义当前项目的模块路径。

- go：用于设置预期的 Go 版本。

- require：用于设置一个特定的模块版本。

- exclude：用于从使用中排除一个特定的模块版本。

- replace：用于将一个模块版本替换为另外一个模块版本。


`go.mod` 文件样例

```
module example.com/foobar

go 1.15

require (
    example.com/apple v0.1.2
    example.com/banana v1.2.3
    example.com/banana/v2 v2.3.4
    example.com/pineapple v0.0.0-20190924185754-1b0db40df49a
)

exclude example.com/banana v1.2.4
replace example.com/apple v0.1.2 => example.com/rda v0.1.0 
replace example.com/banana => example.com/hugebanana
```

通过 `go mod init example.com/m` 命令，会在当前目录下生成一个 `go.mod` 文件

```
module example.com/m

go 1.15
```

当前只包含了模块名，至于依赖会在其它 go 命令运行时自动添加到 `go.mod`

> 自动添加到 `go.mod` 的命令：`go build`, `go test`, or even `go list`

列出当前 module 的依赖

```
go list -m -json all
```

### Go 代理

如果每个人每次下载依赖都直接访问源仓库，不仅会对源仓库的带宽造成很大的压力，而且下载受外部网络影响。

因此，和其它语言的依赖管理工具（比如 maven，npm 等）一样，go modules 也提供了代理支持，可以通过代理服务器来获取依赖，代理服务器会缓存依赖，不用每次都请求仓库

> 公司内部可以自行构建自己的代理服务器

!> Go 代理只有在 module-aware 模式下才有效

默认的代理服务器 `https://proxy.golang.or` 国内无法访问

```bash
$ go env | grep -i goproxy

GOPROXY="https://goproxy.cn,direct"
```

为了解决这个问题，国内某些公司也免费提供了代理服务器，比如

- [goproxy.cn](https://goproxy.cn)

    ```bash
    go env -w GOPROXY=https://goproxy.cn,direct
    ```

- [goproxy.io](https://goproxy.io/zh/)

    ```bash
    go env -w GOPROXY=https://goproxy.io,direct
    ```

> 两个代理服务器都提供了自建服务器方案

### Go Modules 正确玩法

- 配置 GOPROXY

- module_name 使用模块在 VCS 仓库的地址如 `github.com/goproxy/goproxy`

- 使用 replace 实现自引用

> module 内 package 之间以及 module 和子 module 之间的引用是很常见的，但是 Go modules 不支持相对路径引用，只能通过 `module_name/path/to/package` 来引用  
> 如果 module_name 发生变化, 比如换了 VCS 仓库，则引用的地址都要改动，使用 replace 可以解决这个问题

```
module go.etcd.io/etcd/v3

go 1.15

replace (
	go.etcd.io/etcd/api/v3 => ./api
	go.etcd.io/etcd/client/v2 => ./client/v2
	go.etcd.io/etcd/client/v3 => ./client/v3
	go.etcd.io/etcd/etcdctl/v3 => ./etcdctl
	go.etcd.io/etcd/pkg/v3 => ./pkg
	go.etcd.io/etcd/raft/v3 => ./raft
	go.etcd.io/etcd/server/v3 => ./server
	go.etcd.io/etcd/tests/v3 => ./tests
)

require (
	github.com/bgentry/speakeasy v0.1.0
	github.com/dustin/go-humanize v1.0.0
)
```


## References

- [标准库和社区库搜索](https://godoc.org/)

- [标准库和社区库搜索2](https://golang.org/pkg/)

- [干货满满的 Go Modules 和 goproxy.cn](https://juejin.cn/post/6844903954879348750)

- [go modules 使用本地库、公开库和私有库](https://blog.csdn.net/qingchuwudi/article/details/107119273)

- [Golang Package 与 Module 简介](https://www.jianshu.com/p/07ffc5827b26)

- [Golang中的包管理工具 - Go Modules](https://cloud.tencent.com/developer/article/1478299)