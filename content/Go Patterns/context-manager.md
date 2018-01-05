---
title: "Context Manager 上下文管理器 "
date: 2018-01-04T20:12:34-08:00
---

## 定义
Context Manager（上下文管理器）是在不改变编程契约的情况下隐藏控制流的方法。
<!--more-->
```go
package main

import "fmt"

// Context Manager
func withIgnoring7(add func(int, int) int, a int, b int) int {
    r := add(a, b)
    if r == 7 {
        return 0
    }
    return r
}

func add(a int, b int) int {
    return a + b
}

func main() {
    fmt.Println(withIgnoring7(add, 1, 6))
}
```
示例中 `withIgnoring7` 是一个上下文管理器，其作用是忽略等于 7 的值。

上下文管理器有两个要点：  
1. 第一点是不改变被管理函数的接口。`withIgnoring7(add, 1, 6)` 和 `add(1, 6)` 有同样的接口。  
2. 第二点是隐藏控制流。`withIgnoring7` 的用户不知到里面的有那些操作。

## 常用处
### Retry 再尝试
当网络请求出错时，需要重试
```go
package main

import (
	"fmt"
	"errors"
	"time"
)

func WithRetryAfter(seconds int, times int, call func() error) error {
	for i := 0; i < times; i++ {
		err := call()
		if err.Error() == "timeout" {
			fmt.Println(err)
			time.Sleep(time.Second * time.Duration(seconds))
			fmt.Println(i+1, "s")
			continue
		}
		if err != nil {
		        fmt.Println("x")
			return err
		}
		return nil
	}
	return errors.New("max timeout")
}

func request() error {
	return errors.New("timeout")
}

func main() {
	err := WithRetryAfter(1, 3, request)
	if err != nil {
		fmt.Println("x", err)
	}
}
```
复制代码到 https://play.golang.org/

这里是一个假的网络链接超时的例子。这里 `WithRetryAfter(1, 3, request)` 代表重试 3 次，每次等一秒。

### Synchronization 同步
如果我们有一个 semaphore，我们需要管理它的计数。虽然在 Go 里很少直接用到 mutex 和 semaphore，但有时还是很好用的。

首先 Go 语言没有自带 semaphore，需要先实现一个。
```go

```

## 建议
1. 借鉴 Python，用 `withXXX` 或者 `WithXXX` 来命名，让用户知道这是一个上下文管理器。

2. 一定要保证同样的接口，哪怕是 `error` 接口。