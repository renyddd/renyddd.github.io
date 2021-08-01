---
title: "Golang Tips"
date: 2021-07-25T10:38:52+08:00
draft: false
tags: ["golang", "tips"]
categories: ["golang"]
series: ["golang系列"]
---

# golang 小技巧

本文主要记录一些在各种文档中看到的有趣且使用的 golang 小技巧代码片段：

## os.Signal

ref：[https://golang.org/pkg/database/sql/](https://golang.org/pkg/database/sql/)

```golang
func main() {
  // initialization some functions
  ctx, stop := context.WithCancel(context.Background())
	defer stop()

	appSignal := make(chan os.Signal, 3)
	signal.Notify(appSignal, os.Interrupt)

	go func() {
		select {
		case <-appSignal:
			stop()
		}
	}()
  
  // ...
}
```

Tips：

1. stop channel 为什么要设为 buffered 为 3 呢？
2. 可以通过 select 不同的信号，来实现优雅和不优雅的推出方式吗？
