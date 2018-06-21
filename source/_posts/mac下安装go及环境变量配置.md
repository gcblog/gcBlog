---
title: mac下安装go及环境变量配置
categories:
  - go
tags:
  - go
  - 环境变量
date: 2018-04-04 17:10:56
---


以下命令都是直接在根目录下执行即可

### 方法一：homebrew

`homebrew`是Mac系统下面目前使用最多的管理软件的工具，目前已支持Go，可以通过命令直接安装Go,不过在这之前需要先安装`homebrew`

安装命令
```
$ brew update && brew upgrade
$ brew install go
```
安装完输入 `go env` 查看环境信息

```
GOARCH="amd64"
GOBIN=""
GOCACHE="/Users/qianjianeng/Library/Caches/go-build"
GOEXE=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOOS="darwin"
GOPATH="/Users/qianjianeng/go"
GORACE=""
GOROOT="/usr/local/Cellar/go/1.10/libexec"
GOTMPDIR=""
GOTOOLDIR="/usr/local/Cellar/go/1.10/libexec/pkg/tool/darwin_amd64"
GCCGO="gccgo"
CC="clang"
CXX="clang++"
CGO_ENABLED="1"
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=/var/folders/vj/0nsxxzqd3cgdqnwkzqxqg6zr0000gn/T/go-build786549549=/tmp/go-build -gno-record-gcc-switches -fno-common"
```
<!--more-->

### 方法二：pkg包安装

直接去官方下载安装包，然后双击安装,之后同样地输入`go env`、`go version`等查看是否安装。

### 环境变量配置

#### 打开`bash_profile`

`$ open .bash_profile`
#### 如果不存在创建bash_profile
`$ vim ~/.bash_profile`

#### 添加go环境变量

如果是第一种安装方法，只需要指定一下GOPATH即可。为了让自己的程序编译之后在命令行任何地方能直接执行，再加入GOPATH下的bin即可：
```
#This is my personal bash_profile,when loaded at login.
#===2015-08-15===

#GOPATH
export GOPATH=$HOME/Documents/go_workspace

#GOPATH bin
export PATH=$PATH:$GOPATH/bin
```
但是第二种方法安装之后输入go可能会显示ommand not found: go，所以需要在.bash_profile中指定GOROOT下的bin：
```
#This is my personal bash_profile,when loaded at login.
#===2015-08-15===
#GOROOT
export GOROOT=/usr/local/go

#GOPATH
export GOPATH=$HOME/Documents/go_workspace

#GOROOT bin
export PATH=$PATH:$GOROOT/bin

#GOPATH bin
export PATH=$PATH:$GOPATH/bin
```

一般环境变量更改后，重启后生效。在重启终端的时候就会自动执行.bash_profile文件。
如果想立刻生效，则可执行下面的语句：

`$ source .bash_profile`

### 注意

如果打开终端没有生效，就把上面添加go环境变量的语句追加到~/.zshrc中：

`$ open .zshrc`


