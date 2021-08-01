---
title: "Golang Errors"
date: 2021-07-25T19:37:47+08:00
draft: true
tags: ["golang"]
categories: ["golang"]
series: ["golang系列"]
---

# golang error 处理

首先我们来看看 error 类型在 go 语言中的定义：

```golang
/* https://golang.org/pkg/builtin/#error */
// The error built-in interface type is the conventional interface for
// representing an error condition, with the nil value representing no error.
type error interface {
	Error() string
}
```

Error 就是一个接口类型，因此我们总能通过 err != nil 来判断；再来看 errors 包中的 New 方法，所返回的是仅包含有一段描述信息的 errorString 类型的指针，因此每一次对 New 的调用所产生的都是不同值：

```golang
/* https://golang.org/src/errors/errors.go */
// New returns an error that formats as the given text.
// Each call to New returns a distinct error value even if the text is identical.
func New(text string) error {
	return &errorString{text}
}

// errorString is a trivial implementation of error.
type errorString struct {
	s string
}

func (e *errorString) Error() string {
	return e.s
}
```



to be done.
