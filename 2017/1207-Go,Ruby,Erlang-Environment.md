## Go, Ruby, Erlang 环境配置

### Go 介绍

Go 语言是 Google 开发的语言，它被认识是「更好用的 C++」。它的特性主要是：

- 开发速度
  - 利益于它轻快的编译器和简单的依赖处理算法，编程序起来可是飞快
- 并发
  - Go 有强大的并发库：Goroutines, Channel
- 类型系统
  - 类型简单，推崇用组合模式（composition pattern）来实现类
- 内存管理
  - Go 有自带 GC

### Go 的环境配置

https://golang.org/doc/install

和装一般的程序差不多，解压然后配 PATH 就行了。

用程序测试一下：

```go
package main

import "fmt"

func main() {
    fmt.Printf("hello, world\n")
}
```

`go build` 后就会在当前目录下编出一个与目录名相同的可执行文件。

### Go 版本控制

一般是用 gvm：https://github.com/moovweb/gvm

## Ruby 环境配置

Mac 是自带了低版本的 ruby 的。

直接装包管理器，然后再装高版本的 ruby 吧。

Ruby 的包管理器主流的有 RVM 和 rbenv，RVM 是乎会重一点，大家都喜欢后出的 rbenv。这里也就 rbenv 吧。（rbenv 和 RVM 是不兼容的）

项目：https://github.com/rbenv/rbenv

Mac 可以直接 homebrew 装 rbenv 的。

```
brew install ruby-build
```

*装了 ruby-build 才能用 rbenv install 命令。*

安装最新版的 ruby：

```
rbenv install 2.4.2
```

然后用 global 全局启用就行了：

```
rbenv global 2.4.2
```

## Erlang 环境配置

### 介绍

Erlang 一般用来编写大型的可扩展的实时系统。OTP 则是对应的一些库。

### 安装

http://erlang.org/doc/installation_guide/INSTALL.html 官方的文档说得很全，不过也比较麻烦。

http://docs.basho.com/riak/kv/2.2.3/setup/installing/source/erlang 这个会比较容易看懂。

https://github.com/kerl/kerl 也可以用 kerl 来安装。

https://packages.erlang-solutions.com/erlang/ 各个版本的资源。

自己 make 起来还是挺麻烦的。Mac 下可以直接用 homebrew 安装。

### 使用

后面开始直接使用 shell 测试 erlang，shell 里的命令可以用 `help()` 来看。

创建 helloworld.erl

```erlang
% hello world program
-module(helloworld).
-export([start/0]).

start() ->
io:fwrite("Hello World!\n").
```

然后编译：`erl -noshell -s helloworld start -s init stop`

之后运行生成的可执行文件就行了。



