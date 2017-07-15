# defer, panic 与 recover
这小节我们要介绍Go里面的函数执行流程。

### defer
Go语言中有种不错的设计，即延迟（defer）语句，你可以在函数中添加多个defer语句。
当函数执行到最后时，这些defer语句会按照逆序执行，最后该函数返回。
特别是当你在进行一些打开资源的操作时，遇到错误需要提前返回，在返回前你需要关闭相应的资源，不然很容易造成资源泄露等问题。
如下代码所示，我们一般写打开一个资源是这样操作的：
```go
func ReadWrite() bool {
	file.Open("file")
	// 做一些工作
	if failureX {
		file.Close()
		return false
	}

	if failureY {
		file.Close()
		return false
	}

	file.Close()
	return true
}
```

我们看到上面有很多重复的代码，Go的`defer`有效解决了这个问题。使用它后，不但代码量减少了很多，而且程序变得更优雅。
在`defer`后指定的函数会在函数退出前调用。
```go
func ReadWrite() bool {
	file.Open("file")
	defer file.Close()

	if failureX {
		return false
	}

	if failureY {
		return false
	}

	return true
}
```

如果有很多调用`defer`，那么`defer`是采用后进先出模式，所以如下代码会输出`4 3 2 1 0`
```go
for i := 0; i < 5; i++ {
	defer fmt.Printf("%d ", i)
}
```

### Panic和Recover
Go没有像Java那样的异常机制，它不能抛出异常，而是使用了`panic`和`recover`机制。
一定要记住，你应当把它作为最后的手段来使用，也就是说，你的代码中应当没有，或者很少有`panic`的东西。
这是个强大的工具，请明智地使用它。那么，我们应该如何使用它呢？

Panic
> 是一个内建函数，可以中断原有的控制流程，进入一个令人恐慌的流程中。
> 当函数`F`调用`panic`，函数F的执行被中断，但是`F`中的延迟函数会正常执行，然后F返回到调用它的地方。
> 在调用的地方，`F`的行为就像调用了`panic`。
> 这一过程继续向上，直到发生`panic`的`goroutine`中所有调用的函数返回，此时程序退出。
> 恐慌可以直接调用`panic`产生。也可以由运行时错误产生，例如访问越界的数组。

Recover
> 是一个内建的函数，可以让进入令人恐慌的流程中的`goroutine`恢复过来。
> `recover`仅在延迟函数中有效。在正常的执行过程中，调用`recover`会返回`nil`，并且没有其它任何效果。
> 如果当前的`goroutine`陷入恐慌，调用`recover`可以捕获到`panic`的输入值，并且恢复正常的执行。

下面这个函数演示了如何在过程中使用`panic`
```go
var user = os.Getenv("USER")

func init() {
	if user == "" {
		panic("no value for $USER")
	}
}
```

下面这个函数检查作为其参数的函数在执行时是否会产生`panic`：
```go
func throwsPanic(f func()) (b bool) {

	defer func() {
		if x := recover(); x != nil {
			b = true
		}
	}()

	f() //执行函数f，如果f中出现了panic，那么就可以恢复回来

	return
}
```

### 进阶
下面看一个示例：
```go
package main

import (
	"fmt"
)

func f1() {
	defer func() {
		if x := recover(); x != nil {
			fmt.Println("Recover:", x)
		}
	}()
	fmt.Println("f1")
	defer func() {
		fmt.Println("defer before f2")
	}()
	f2()
	defer func() {
		fmt.Println("defer after f2")
	}()
	fmt.Println("after f2")
}

func f2() {
	fmt.Println("f2")
	panic("f2")
}

func main() {
	f1()
}
```
这个Demo最终的输出会是什么呢？

要弄清楚输出的内容及顺序，就得了解`defer`, `panic`及`recover`之间的关系。这里先给出上面Demo输出的结果：
```
f1
f2
defer before f2
Recover: f2
```
根据上面结果可以得知：

1. `panic`后的所有代码都不会再执行了，即使通过`recover`进行了恢复。
2. 在`recover`之前，**已经**`defer`过的代码块会按倒序进行执行(也就是`panic`后的`defer`也不会执行)。
3. 在`recover`之后，函数就执行了`return`操作，也就是不会再执行当前函数里的代码了。
