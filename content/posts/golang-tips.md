---
title: "Golang Tips"
date: 2021-07-25T10:38:52+08:00
draft: false
tags: ["golang", "tips"]
categories: ["golang"]
series: ["golang系列"]
---

# golang 小技巧

本文主要记录一些在各种文档中看到的有趣且使用的 golang 小技巧代码片段。

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



## logrus

> [https://github.com/sirupsen/logrus](https://github.com/sirupsen/logrus)

Logrus encourages careful, structured logging through logging fields instead of long, unparseable error messages. For example, instead of: `log.Fatalf("Failed to send event %s to topic %s with key %d")`, you should log the much more discoverable:

```golang
log.WithFields(log.Fields{
  "event": event,
  "topic": topic,
  "key": key,
}).Fatal("Failed to send event")
```

We've found this API forces you to think about logging in a way that produces much more useful logging messages. We've been in countless situations where just a single added field to a log statement that was already there would've saved us hours. The `WithFields` call is optional.

In general, with Logrus using any of the `printf`-family functions should be seen as a hint you should add a field, however, you can still use the `printf`-family functions with Logrus.



https://github.com/davecgh/go-spew



## Mock

ref:

- gomock tutorial: https://blog.codecentric.de/en/2017/08/gomock-tutorial/

- another GoMock tutoria（可提炼一些原理）l: https://betterprogramming.pub/a-gomock-quick-start-guide-71bee4b3a6f1

> Note: Both frameworks can only mock for interfaces, so we need to organize code dependencies behind interfaces. This will benefit our unit testing even when no mocking is involved.



https://www.telerik.com/products/mocking/unit-testing.aspx

> Mocking is a process used in unit testing when the unit being tested has external dependencies. The purpose of mocking is to isolate and focus on the code being tested and not on the behavior or state of external dependencies. 



一些不使用 mock 的原因 https://www.accenture.com/us-en/blogs/software-engineering-blog/to-mock-or-not-to-mock-is-that-even-a-question

mock without mocking framework： https://medium.com/@ankur_anand/how-to-mock-in-your-go-golang-tests-b9eee7d7c266



## func WithTimeout

```golang
func simulateWithTimeout(f func(), duration time.Duration) func() {
	return func() {
		ctx, cancel := context.WithTimeout(context.TODO(), duration)
		defer cancel()

		pDone := make(chan struct{})

		go func() {
			f()
			pDone <- struct{}{}
		}()

		select {
		case <-ctx.Done():
			log.Println("process timeout")
		case <-pDone:
			log.Println("process done in time")
		}
	}
}

func main() {
	simulateWithTimeout(func() {
		i := 0
		for {
			i++
		}
	}, time.Second)()

	simulateWithTimeout(func() {
		i := 0
		for i < 10 {
			i++
		}
	}, time.Second)()
}

```



like:

- http.Client timeout
- gin context implement





