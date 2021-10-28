---
title: Goland配置项目moby、containerd和runc
tags:
  - IDE
  - Cloud Native
  - Code Audit
  - Go
categories: Misc
date: 2021-10-28 22:27:23
---


## 0. 前言
如果想要好好审计代码，能够查找引用和声明是必不可少的，如果用普通的编辑器并且依靠搜索来寻找引用和声明，不是说不行，但除非是超级老手，一般效率都比不过Ctrl+左键直接点。

个人是JetBrains吹，所以就喜欢JetBrains全家桶审计代码。

这一篇会介绍怎么用Goland对moby、containerd和runc项目配置，整体其实非常简单，containerd和runc都用了module机制，唯一有点烦的moby只需要对项目进行改名就行了。

## 1. 获取代码
```bash
# disable GO111MODULE so that go get will download source code to GOPATH
export GO111MODULE=off  
# set GO111MODULE=off # for Windows
go get -d github.com/moby/moby
go get -d github.com/containerd/containerd
go get -d github.com/opencontainers/runc  # 这里会碰到报没开cgo的错，暂时不清楚会有什么后果
```
全部跑完以后代码就在GOPATH下面了

## 2. 修改moby项目的文件夹名
由于moby曾经的名字是docker，直到2017年项目名称docker才被改成了moby，为了保持上下游项目的兼容，项目代码中的import路径仍然沿用了docker。

与其克隆一份docker/docker的源码放在GOPATH下面，不如直接把moby/moby源码目录改成docker/docker来方便Goland寻找依赖。

Linux:
```bash
mkdir $GOPATH/src/github.com/docker
mv $GOPATH/src/github.com/moby/moby $GOPATH/src/github.com/docker/docker
```

Windows Powershell:
```bash
mkdir $env:GOPATH\src\github.com\docker
mv $env:GOPATH\src\github.com\moby\moby $env:GOPATH\src\github.com\docker\docker
```

## 3. 配置docker(moby)项目
使用Goland的Open打开docker/docker的目录以后，需要检查settings-Go（导航栏第一个）中的这些配置：
- `GOROOT`配置中选择Go的SDK
- `Go Modules`中确保`Enable go modules integration`是**关闭**的
- `Build Tags & Vendoring`中`Editor Constraints`部分选择合适的OS，通常选linux（需要配置完GOROOT后重新进入设置）

完成以后就可以享受`Ctrl+左键`和`Ctrl+Alt+H`了

## 4. 配置containerd项目和runc项目
containerd和runc没那么多幺蛾子，直接用了go modules，所以只要用Goland的Open打开对应的目录，之后检查settings-Go（导航栏第一个）中的这些配置
- `GOROOT`配置中选择Go的SDK
- `Go Modules`中确保`Enable go modules integration`是**打开**的，有需要可以在`Environment`中配置GOPROXY，如`GOPROXY=https://mirrors.aliyun.com/goproxy/`
- `Build Tags & Vendoring`中`Editor Constraints`部分选择合适的OS，通常选linux（需要配置完GOROOT后重新进入设置）

## 5. 完成
至此就大功告成了，可以充分享受Goland的特性了xD
