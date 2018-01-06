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
虽然在 Go 里很少直接用到 mutex，但有时还是很好用的。

通常可以这样写
```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var mutex = &sync.Mutex{}
	mutex.Lock()
	fmt.Println("Lock")
	fmt.Println("Do something")
	a := 1
	b := 2
	c := a + b
	fmt.Println("Unlock")
	mutex.Unlock()
	fmt.Println("c =", c)
}
```
我们可以用一个 `WithSynchronizedAccess` 来包裹。
```go
package main

import (
	"fmt"
	"sync"
)

func WithSynchronizedAccess(mutex *sync.Mutex, f func() int) int {
	mutex.Lock()
	fmt.Println("Lock")
	defer func() {
		fmt.Println("Unlock")
		mutex.Unlock()
	}()
	return f()
}

func main() {
	var mutex = &sync.Mutex{}
	f := func() int{
		fmt.Println("Do something.")
		a := 1
		b := 2
		c := a + b
		return c
	}
	c := WithSynchronizedAccess(mutex, f)
	fmt.Println("c =", c)
}
```
复制代码到 https://play.golang.org/

这里我们用一个 local 函数将之前散装的代码打包了。如果你喜欢 inline 风格也可以。
```go
func main() {
	var mutex = &sync.Mutex{} 
	c := WithSynchronizedAccess(mutex, func() int {
		fmt.Println("Do something.")
		a := 1
		b := 2
		c := a + b
		return c
	})
	fmt.Println("c =", c)
}
```
## 建议（从重要到次要）
1. 一定要保证同样的接口，哪怕是 `error` 接口。

2. 函数签名用 `withXXX(paramsOfCM..., fun, paramsOfFun...)`  
	| 被管理函数前面的参数为内容管理器自身的参数，之后的参数为被管理函数的参数。

3. 借鉴 Python，用 `withXXX` 或者 `WithXXX` 来命名，让用户知道这是一个上下文管理器。
