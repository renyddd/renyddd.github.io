---
title: "Golang Mod"
date: 2021-06-21T17:31:12+08:00
draft: false
tags: ["golang"]
categories: ["golang"]
series: ["golang系列"]
---

## go mod 与 git tag

### 报错跟踪记录

因笔者想使用某仓库的某一函数（xxxpkg.Hello 为例），该仓库已通过 git clone 至本地，通过独立的 ide 窗口进行查看。我自己的项目由另一 ide 窗口进行开发，在该项目中始终无法引用该函数。

但当通过 master 版本进行更新后，即可于本项目中引用该函数。

```go
cat go.mod
// github.com/xxx/xxxpkg v1.0.1

go get github.com/xxx/xxxpkg@master
go: github.com/xxx/xxxpkg master => v1.0.2-0.20210621053006-47178e58dbcf
```

注意，该 go.mod 文件中的 pkg 版本为一哈希值，而非指定版本。

而后笔者为了解决 [etcd 依赖问题](https://github.com/etcd-io/etcd/issues/11931)时，对 grpc 包进行了指定版本更新后，此次更新又触发了所需 xxxpkg 版本的变更。

即如下操作又导致了 xxxpkg 对 grpc@v1.26.0 依赖的版本更新。

```go
go get google.golang.org/grpc@v1.26.0                                                                                                    2
bash build.sh && ./main              
go: finding module for package github.com/xxx/xxxpkg
go: found github.com/xxx/xxxpkg in github.com/xxx/xxxpkg v1.0.1
# github.com/xxx/xxxpkg
xxx.go:17:12: undefined: xxxpkg.Hello

```

最终解决原因定位为，github.com/xxx/xxxpkg 远端仓库已有的 tag v1.0.2 已是两个月前，并未包含 master 分治的最新代码。通过打新 tag 版本即可最终解决。

### go mod



### git tag
