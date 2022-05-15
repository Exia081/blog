---
title: GO错误异常处理 
date: 2022-05-14 13:50:58 
categories:
- GO语言 
tags:
- GO语言
---

# 错误处理

在GO语言中处理错误，通过接口来处理

```go
package builtin

type error interface {
	Error() string
}
```

在其他语言中，错误有可能是错误码，有可能是错误信息

错误处理非常重要，go语言将其统一成为接口，如果需要获取错误的信息，则调用error()接口，返回错误信息字符串，这统一了错误信息，都作为字符串输出

## 问题

在错误处理的流程中，在其他语言中类似java，提供了try-catch-finally方式来处理，但go的err则会编写一大串对应的错误处理代码，显得非常冗长

```go
package main

func main() {
	err := doSomething()
	if err != nil {
		//handle error ...
	}

	err = doSomething2()
	if err != nil {
		//handle error ...
	}

	err = doSomething3()
	if err != nil {
		//handle error ...
	}
}
```

这些问题在go的作者群体中，也产生过大量的讨论和修改建议，但目前没有形成统一的新的处理方案

FAQ里面也有提到 try-catch 会让代码变得非常混乱，对于真正的异常，作者希望通过panic-recover机制进行处理，这样代码看起来更简洁

```go
package main

func main() {
	defer func() {
		if err := recover(); err != nil {
			//handle panic ...
		}
	}()

	err := doSomething()
	if err != nil {
		//handle error ...
	}
}
```

## 如何理解error

事实上，作者们已经形成了很多醒世名言，下面抽选关于错误和异常相关的进行解释

```
Errors are just a value.
Don't just check errors,handle them gracefully.
Don't panic.
```

### Errors are just a value.(错误只是一种值)

事实上错误的处理可分为三种情况：
* Sentinel errors （哨兵错误）
* Error Types （类型错误）
* Opaque errors （黑盒错误）

###### Sentinel errors （哨兵错误）

通常情况下，出现某种错误，程序流程就不能继续往下执行了

但在实际使用过程中，往往会定义很多对应的哨兵错误，并且业务代码与它强耦合，这会导致引用很多包的时候，他们的错误意义是接近的，但却要写出大量意义相近的错误处理流程

```go
if  err == io.EOF {
//todo 
} else if err == io.ErrUnexpectedEOF {
//todo 
}
```

###### Error Types （类型错误）

它指的是实现了错误接口的类型错误，其中可以附带其他字段，提供更多信息，如

```
type BussinessError struct {
    BusinessCode int
    Caller string
    Err error
}
```

在这个错误中，我们额外增加的 业务code，和调用者 等信息

通常在这种情况的错误处理，我们需要使用**类型断言**来进行处理，如

```
func handleErr(err error) string {
	switch err := err.(type) {
	case BusinessError:
		return err.Error()
	case DBError:
		return err.Error()
	case CacheError:
		return err.Error()
	}
	return ""
}
```

与哨兵错误类似，业务代码与它强耦合，增加了更多信息，方便根据信息处理不同的情况，减少了哨兵错误定义的数量，但大量错误逻辑仍然是需要

###### Opaque errors （黑盒错误）

最后一种则是最直接的，不管接收到什么错误，都直接返回错误

```
if err := doSomething(); err != nil {
    return err
}
```

### Don't just check errors,handle them gracefully.（优雅地处理错误）

在GO语音中，提供了很多对错误进行优化处理的方案

```
//对error信息进行包装,并且带上调用栈信息
errors.Wrap(err error, message string) error {}
//按格式进行包装,并且带上调用栈信息
errors.Wrapf(err error, format string, args ...interface{}) error
//对error进行解包，实现cause接口的error
errors.Cause(err error) error 
```

### Only handle error once（只处理错误一次）

在业务代码中，常常会写出很深的调用链

这时候如果每一层都对错误进行处理，如打印日志，则打印大量相同的日志

因此在GO的设计中，它希望的是最上层调用者进行处理，底层捕获的error，对其进行warp，这样就能暴露调用链，同时也不会在各个层级产生大量相同的错误处理代码

### Don't Panic (不要滥用异常)

在GO语言的设计中，每个Gorutines都是独立的，因此并没有父子Gorutines的说法，更不能捕获其他gorutine的panic

在一个gorutine中没有设置defer recover函数，发生panic则后果是灾难，会导致整个go程序退出

**在GO语言的作者中认为，只有发生不可挽回的事情，才需要触发panic，避免事故影响扩大**

**如果把panic当做error使用，则会导致无法正常区分哪些是灾难异常，哪些是业务错误**

# 后续

事实上，go目前的错误处理并不完美，在作者群体中，对go的错误处理也有广泛的讨论，并且把它作为go 2 版本的优化重点

我们可以对官方信息保持关注

[Go 2 官方草案](https://go.googlesource.com/proposal/+/master/design/go2draft.md "Go 2 官方草案")。

