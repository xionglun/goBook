# Opentracing
[Opentracing](http://opentracing.io/)是一个调用链路追踪规范，它类似于[Zipkin](http://zipkin.io/)和[Google Dapper](https://research.google.com/pubs/pub36356.html)。

与Zipkin不同的是，它定义了协议，并提供了多种语言的客户端库，但是没有提供最终存储和展示的实现。
用户可以自定义对接到不同的后端兼容层上，只要其兼容于Opentracing协议即可。

如果你想了解Opentracing协议具体内容，可以参考[Opentracing中文文档](https://github.com/opentracing-contrib/opentracing-specification-zh)


### 使用Opentracing
目前比较成熟的开源调用追踪存储展示应用是Zipkin，而符合Opentracing协议且比较全面覆盖的是[Uber Jaeger](https://uber.github.io/jaeger)。

目前Opentracing官方客户端支持[Go](https://github.com/opentracing/opentracing-go), [Python](https://github.com/opentracing/opentracing-python), [Javascript](https://github.com/opentracing/opentracing-javascript), [Java](https://github.com/opentracing/opentracing-java), [C#](https://github.com/opentracing/opentracing-csharp), [Objective-C](https://github.com/opentracing/opentracing-objc), [C++](https://github.com/opentracing/opentracing-cpp), [Ruby](https://github.com/opentracing/opentracing-ruby)这八种语言。同时，PHP的支持正在进行中。


#### 存储到Zipkin
由于Zipkin是Twitter公司使用Java语言开发的，同时也主要用于Java应用，所以对Java支持比较好，对其它语言支持一般。
同时，由于Zipkin是诞生在Opentracing协议之前，所以对Opentracing兼容性并不是很好。

但是，由于Zipkin的[Golang库](https://github.com/openzipkin/zipkin-go-opentracing)起步比较晚，所以遵循了Opentracing协议。

示例：
```go
package main

import (
	opentracing "github.com/opentracing/opentracing-go"
	zipkin "github.com/openzipkin/zipkin-go-opentracing"
)

func initTracing() {
	endpoint := "" // Zipkin的地址
	debug := false // 是不是debug模式
	hostPort := "192.168.2.11:8080" // 当前服务的host,port
	svcName := "ServiceName" // 当前服务名称

	collector, err := zipkin.NewHTTPCollector(endpoint)
	if err != nil {
		log.Printf("unable connect to collector: %s, %+v\n", endpoint, err)
	} else {
		// create recorder.
		recorder := zipkin.NewRecorder(collector, debug, hostPort, svcName)
		// create tracer.
		tracer, err := zipkin.NewTracer(recorder)
		if err != nil {
			log.Printf("unable to create Zipkin tracer: %+v", err)
		} else {
			opentracing.InitGlobalTracer(tracer)
		}
	}
}

func init() {
	initTracing()
}

func invokeService(span opentracing.Span) {
	sp := opentracing.StartSpan("Invoke Service", opentracing.ChildOf(span.Context()))
	defer sp.Finish()
	sp.SetTag("Service", "service name")
	// ... 其它调用
}

func Handler(w http.ResponseWriter, r *http.Request) {
	var span opentracing.Span
	// 从上一个调用服务获取调用链信息
	spCtx, err := tracing.GlobalTracer().Extract(opentracing.HTTPHeaders, tracing.HTTPHeadersCarrier(r.Header))
	if err != nil {
		span = tracing.StartSpan(spanName)
	} else {
		span = tracing.StartSpan(spanName, tracing.ChildOf(spCtx))
	}
	defer span.Close()
	invokeService(span)
	// 其它处理
}
```

#### 存储到Jaeger
Jaeger是Uber公司工程师根据Opentracing协议编写的存储后端。官方提供了[Go](https://github.com/uber/jaeger-client-go), [Java](https://github.com/uber/jaeger-client-java), [Python](https://github.com/uber/jaeger-client-python), [Node.js](https://github.com/uber/jaeger-client-node) 这四个库。

Jaeger对Opentracing支持力度比Zipkin要大，但由于其出现比较晚，所以未能有很广泛的应用（主要是在Uber内部使用）。
同时，其针对分布式大规模应用有优化，所以架构复杂度上要比Zipkin稍微复杂。
