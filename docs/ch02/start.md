# Go语言基础

Go是一门类似C的编译型语言，但是它的编译速度非常快。
这门语言的关键字总共也就二十五个，比英文字母还少一个，这对于我们的学习来说就简单了很多。
先让我们看一眼这些关键字都长什么样：
```
break    default      func    interface    select
case     defer        go      map          struct
chan     else         goto    package      switch
const    fallthrough  if      range        type
continue for          import  return       var
```

- var和const参考 [Go语言基础](./base.md)里面的变量和常量申明
- package和import已经有过短暂的接触
- func 用于定义函数和方法
- return 用于从函数返回
- defer 用于类似析构函数
- go 用于并发
- select 用于选择不同类型的通讯
- interface 用于定义接口，参考 [interface](./interface.md)小节
- struct 用于定义抽象数据类型，参考 [struct](./struct.md)小节
- break、case、continue、for、fallthrough、else、if、switch、goto、default 参考:[流程介绍](./function.md)
- chan用于channel通讯
- type用于声明自定义类型
- map用于声明map类型数据
- range用于读取slice、map、channel数据

在接下来的这一章中，我将带领你去学习这门语言的基础。
通过每一小节的介绍，你将发现，Go的世界是那么地简洁，设计是如此地美妙，编写Go将会是一件愉快的事情。
等回过头来，你就会发现这二十五个关键字是多么地亲切。

