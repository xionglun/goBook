#应用日志
我们期望开发的Web应用程序能够把整个程序运行过程中出现的各种事件一一记录下来，Go语言中提供了一个简易的log包，
我们使用该包可以方便的实现日志记录的功能，这些日志都是基于fmt包的打印再结合panic之类的函数来进行一般的打印、抛出错误处理。
Go目前标准包只是包含了简单的功能，如果我们想把我们的应用日志保存到文件，然后又能够结合日志实现很多复杂的功能
(编写过Java或者C++的读者应该都使用过log4j和log4cpp之类的日志工具),
可以使用Uber开发的一个日志系统，[Uber zap](https://github.com/uber-go/zap)，它主要关注两个方面：性能与易用性。
接下来我们介绍如何通过该日志系统来实现我们应用的日志功能。


## zap 介绍
zap 是用Go语言实现的一个日志系统，它的简介如下：

> Blazing fast, structured, leveled logging in Go.

从这短短的一句话，可以看出，它主要是关注速度，结构化日志，多级别化日志。所以，从它关注点出发，它提供了两种使用方式：
`zap.Logger`和`zap.SugaredLogger`。

初接触`zap`的人可能不明白为什么会有两种不同的接口来使用。但其实它分别对应两种关注点：
- `zap.Logger`提供高性能日志记录，但是它提供的功能函数很少，需要你自行处理一些复杂的日志。
- `zap.SugaredLogger`提供多功能函数的易用日志记录处理，性能相对稍弱，但也比大部份 Go 日志库要好。


### 使用 zap
导入 zap 库
```sh
import "go.uber.org/zap"
```

简单示例：
```go
package main

import (
	"time"

	"go.uber.org/zap"
)

func main() {
	sugar := zap.NewExample().Sugar() // 新建一个 SugaredLogger 对象
	defer sugar.Sync()   // 保证在程序退出时，所有日志都落盘了。
	sugar.Infow("failed to fetch URL",
		"url", "http://example.com",
		"attempt", 3,
		"backoff", time.Second,
	)
	sugar.Infof("failed to fetch URL: %s", "http://example.com")
}
```
打印结果如下：
```
{"level":"info","msg":"failed to fetch URL","url":"http://example.com","attempt":3,"backoff":"1s"}
{"level":"info","msg":"failed to fetch URL: http://example.com"}
```


### zap.Logger
`zap.Logger` 是高性能日志接口，速度非常快，内存占用非常小。

`zap.Logger` 限制比较多，比如没有 Xxxf() 之类的方法，不能格式化待打印的字符串。
但是你可以添加多个`Field`来控制打印出来的日志。如下所示：
```go
logger := zap.NewExample()
defer logger.Sync()
logger.Info("Hello world!", zap.String("lang", "golang"), zap.Int("age", 21))
// {"level":"info","msg":"Hello world!","lang":"golang","age":21}
```

同时，还有一些方法，比如`Named`和`With`等，可以让你输出日志时更清晰明了。如下：
```go
logger := zap.NewExample()
defer logger.Sync()
logger = logger.Named("example").With(zap.String("env", "dev"), zap.String("version", "1.1.0"))
logger.Info("Hello world!", zap.String("lang", "golang"), zap.Int("age", 21))
// {"level":"info","logger":"example","msg":"Hello world!","env":"dev","version":"1.1.0","lang":"golang","age":21}
```


### zap.SugaredLogger
`zap.SugaredLogger` 是通用性多功能日志接口，速度也很快，内存占用相对其它日志库也很小。

`zap.SugaredLogger` 提供了与其它日志库类似的功能，可以使用 Xxxf() 之类的方法。如下：
```go
logger := zap.NewExample()
defer logger.Sync()
logger = logger.Named("example").With(zap.String("env", "dev"), zap.String("version", "1.1.0"))
sugar := logger.Sugar().Named("Bingo").With(zap.String("func", "DoSomething"))
sugar.Infof("Hello, %s", "Jack")
// {"level":"info","logger":"example.Bingo","msg":"Hello, Jack","env":"dev","version":"1.1.0","func":"DoSomething"}
```


### 两种日志接口转换
虽然 `zap` 提供了两种 `Logger` 对象，但是它们可以很轻松地进行转换：
```go

logger, _ := zap.NewProduction()  // 这里 logger 是 *zap.Logger 类型的
defer logger.Sync()

sugar := logger.Sugar()  // 这里 sugar 就是 *zap.SugaredLogger 类型了
mlog := sugar.Desugar()  // 这里 mlog 就是 *zap.Logger 类型了
```


### 输出日志到文件
上面示例中，都是直接终端输出了，并没有写入到日志文件。这里简单介绍一下如下配置日志文件路径：
```go
func NewLogger() (*zap.Logger, error) {
	cfg := zap.NewProductionConfig()
	cfg.OutputPaths = []string{   // 标准输出，一般只需要配置这个即可
		"stdout",  // 同时输出到终端，若不需要，删除此行即可
		"/var/log/example/example.log",  // 输出到文件
	}
	cfg.ErrorOutputPaths = []string{   // 应用出错时输出，比如程序 panic 了
		"stderr",
		"/var/log/example/example.error.log",
	}
	cfg.Level = zap.NewAtomicLevelAt(zap.InfoLevel)
	cfg.Sampling = nil  // 不使用采样，全量打印日志。默认通过采样来输出日志，以防止大量相同日志（如错误日志）输出
	cfg.Encoding = "json"  // 输出为 JSON 格式
	cfg.EncoderConfig = zap.NewProductionEncoderConfig()  // 使用默认生产环境日志编码设置
	return cfg.Build()
}

func main() {
	logger, _ := NewLogger()
	defer logger.Sync()
}
```


### 动态调节日志级别
在开发阶段，一般会把应用程序日志级别设置成 **debug** 级别，而到了生产环境，会将其设置成 **info** 或者更高级别。  
但是，在某些时候，线上程序出现问题，需要打印一段时间的详细日志，这样就需要将日志设置成 **debug** 比较方便。




### 小结
本文只是简单介绍了 `zap` 这个日志库的使用。如果需要进一步了解如何使用，请参考官网文档: [Godoc](https://godoc.org/go.uber.org/zap)


### 参考
1. [从Go高性能日志库zap看如何实现高性能Go组件](https://mp.weixin.qq.com/s/i0bMh_gLLrdnhAEWlF-xDw)
