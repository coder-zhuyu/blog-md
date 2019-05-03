---
title: golang中的error处理
date: 2019-05-02 21:00:35
categories: golang
tags: [golang]
---
# error接口
error类型本身就是一个预定义好的接口， 可以自定义error结构体，只需要实现error接口。

```golang
type error interface {
    Error() string
}
```

errors包

```golang
// Package errors implements functions to manipulate errors.
package errors

// New returns an error that formats as the given text.
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

# 自定义error
```golang
type Error struct {
	Id     string `json:"id"`
	Code   int32  `json:"code"`
	Detail string `json:"detail"`
	Status string `json:"status"`
}

func (e *Error) Error() string {
	b, _ := json.Marshal(e)
	return string(b)
}

// New generates a custom error.
func New(id, detail string, code int32) error {
	return &Error{
		Id:     id,
		Code:   code,
		Detail: detail,
		Status: http.StatusText(int(code)),
	}
}
```

# error附加信息
虽然Go语言对错误的设计非常简洁，但是对于我们开发者来说，很明显是不足的，比如我们需要知道出错的更多信息，在什么文件的，哪一行代码？进一步知道调用的堆栈，只有这样我们才更容易的定位问题。

如果要解决以上的问题，那么首先我们必须再继续扩充我们的errorString，再增加一些字段来存储更多的信息。比如我们要记录堆栈信息。github.com/pkg/errors这个库提供了对应的解决方案。

```golang
type stack []uintptr
type errorString struct {
	s string
	*stack
}

func callers() *stack {
	const depth = 32
	var pcs [depth]uintptr
	n := runtime.Callers(3, pcs[:])
	var st stack = pcs[0:n]
	return &st
}

func New(text string) error {
	return &errorString{
		s:   text,
		stack: callers(),
	}
}

// WithStack annotates err with a stack trace at the point WithStack was called.
// If err is nil, WithStack returns nil.
func WithStack(err error) error {
	if err == nil {
		return nil
	}
	return &withStack{
		err,
		callers(),
	}
}

type withStack struct {
	error
	*stack
}
```

github.com/pkg/errors提供了几个函数可以选择：
```golang
//只附加新的信息
func WithMessage(err error, message string) error

//只附加调用堆栈信息
func WithStack(err error) error

//同时附加堆栈和信息
func Wrap(err error, message string) error
```
