#Go 测试
本小节主要讲解Go语言中如何来实现**单元测试**和**性能测试**。

Go语言中自带有一个轻量级的测试框架`testing`和自带的`go test`命令来实现单元测试和性能测试。   
`testing`框架和其他语言中的测试框架类似，可以基于这个框架写针对相应函数的测试用例，也可以基于该框架写相应的压力测试用例。


## 单元测试
使用`go test`命令只能在一个目录下执行所有测试文件，如果想要执行当前目录及递归子目录所有测试文件，可以使用`go test ./...`，注意后面是三个点。

我们在同一目录下面创建两个文件：gotest.go和gotest_test.go   
1. gotest.go:这个文件里面有一个函数`Division`实现了除法运算:
  ```go
	package gotest
		
	import (
		"errors"
	)
		
	func Division(a, b float64) (float64, error) {
		if b == 0 {
			return 0, errors.New("除数不能为0")
		}
		
		return a / b, nil
	}
  ```

2. gotest_test.go: 这是针对上面函数的单元测试文件，它遵循下面的这些原则：   
	- 文件名必须以`_test.go`结尾
	- 文件必须`import "testing"`这个**testing**包
	- 所有的测试用例函数必须以`Test`开头
	- 测试用例会按照源代码中写的顺序依次执行
	- 测试函数`TestXxx()`的参数是`testing.T`，可以使用该类型来记录错误或者是测试状态
	- 测试函数格式：`func TestXxx (t *testing.T)`,`Xxx`部分可以为任意的字母数字的组合，但是首字母必须是大写字母[A-Z]，例如`Testintdiv`是错误的函数名。
	- 函数中通过调用`testing.T`的`Error`, `Errorf`, `FailNow`, `Fatal`, `FatalIf`方法来表明测试未通过，调用`Log`方法用来记录测试的信息。
	
	单元测试代码示例：
	```go
	package gotest
		
	import (
		"testing"
	)
		
	func Test_Division_1(t *testing.T) {
		if i, e := Division(6, 2); i != 3 || e != nil { //try a unit test on function
			t.Error("除法函数测试没通过") // 如果不是如预期的那么就报错
		} else {
			t.Log("第一个测试通过了") //记录一些你期望记录的信息
		}
	}
		
	func Test_Division_2(t *testing.T) {
		t.Error("就是不通过")
	}
  ```
	在项目目录下面执行`go test`,就会显示如下信息：
	```
		--- FAIL: Test_Division_2 (0.00 seconds)
			gotest_test.go:16: 就是不通过
		FAIL
		exit status 1
		FAIL	gotest	0.013s
	```
	结果显示测试没有通过，因为在第二个测试函数中有测试不通过的代码`t.Error`。   
	如果想显示测试过程中具体的信息，需要带上参数执行`go test -v`，这样就会显示如下信息：
	```
		=== RUN Test_Division_1
		--- PASS: Test_Division_1 (0.00 seconds)
			gotest_test.go:11: 第一个测试通过了
		=== RUN Test_Division_2
		--- FAIL: Test_Division_2 (0.00 seconds)
			gotest_test.go:16: 就是不通过
		FAIL
		exit status 1
		FAIL	gotest	0.012s
	```
	上面的输出详细的展示了这个测试的过程，可以看到测试函数1`Test_Division_1`测试通过，而测试函数2`Test_Division_2`测试失败了，
	最后结论测试不通过。   
	如果把测试函数2修改成如下代码：
	```go
	func Test_Division_2(t *testing.T) {
		if _, e := Division(6, 0); e == nil { //try a unit test on function
			t.Error("Division did not work as expected.") // 如果不是如预期的那么就报错
		} else {
			t.Log("one test passed.", e) //记录一些你期望记录的信息
		}
	}
	```
	然后我们执行`go test -v`，就显示如下信息，测试通过了：
	```
	=== RUN Test_Division_1
	--- PASS: Test_Division_1 (0.00 seconds)
		gotest_test.go:11: 第一个测试通过了
	=== RUN Test_Division_2
	--- PASS: Test_Division_2 (0.00 seconds)
		gotest_test.go:20: one test passed. 除数不能为0
	PASS
	ok  	gotest	0.013s
  ```


#### Tips
1. 测试单个文件
  
    go test -v hello_test.go hello.go
  
  注意此处，如果要测试某一个文件，需要把测试文件与原文件都列出来

2. 测试某一个函数
  
    go test -v -test.run TestLogin
  
  这里测试`TestLogin`这个测试用例，只需要列出该函数名即可。


## 压力测试
压力测试用来检测函数(方法）的性能，形式和编写单元功能测试的方法类似，但需要注意以下几点：   
- 压力测试用例必须遵循如下格式，其中XXX可以是任意字母数字的组合，但是首字母不能是小写字母
  ```go
	func BenchmarkXXX(b *testing.B) { ... }
	```
- `go test`不会默认执行压力测试的函数，如果要执行压力测试需要带上参数`-test.bench`，语法:`-test.bench="test_name_regex"`,
  例如`go test -test.bench=".*"`表示测试全部的压力测试函数。
- 在压力测试用例中,请记得在循环体内使用`testing.B.N`,以使测试可以正常的运行。
- 文件名也必须以`_test.go`结尾。

新建一个压力测试文件webbench_test.go，代码如下所示：
```go
package gotest
	
import (
	"testing"
)
	
func Benchmark_Division(b *testing.B) {
	for i := 0; i < b.N; i++ { //use b.N for looping 
		Division(4, 5)
	}
}
	
func Benchmark_TimeConsumingFunction(b *testing.B) {
	b.StopTimer() //调用该函数停止压力测试的时间计数

	//做一些初始化的工作,例如读取文件数据,数据库连接之类的,
	//这样这些时间不影响我们测试函数本身的性能
	
	b.StartTimer() //重新开始时间
	for i := 0; i < b.N; i++ {
		Division(4, 5)
	}
}
```

执行命令`go test -file webbench_test.go -test.bench=".*"`，可以看到如下结果：
```
PASS
Benchmark_Division	500000000	         7.76 ns/op
Benchmark_TimeConsumingFunction	500000000	         7.80 ns/op
ok  	gotest	9.364s	
```
上面的结果显示没有执行任何`TestXXX`的单元测试函数，显示的结果只执行了压力测试函数，
第一条显示了`Benchmark_Division`执行了500000000次，每次的执行平均时间是7.76纳秒，
第二条显示了`Benchmark_TimeConsumingFunction`执行了500000000，每次的平均执行时间是7.80纳秒。
最后一条显示总共的执行时间。

