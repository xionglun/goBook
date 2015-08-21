# 交叉编译
Go是一门编译型语言，所以在不同平台上，需要编译生成不同格式的二进制包。

由于Go 1.5对跨平台编译有了一些改进，包括统一了编译器、链接器等。   
编译时候只需要指定两个参数：`GOOS`和`GOARCH`即可。

示例：
```sh
# 编译到 linux 64bit
$ GOOS=linux GOARCH=amd64 go build
# 或者可以使用 -o 选项指定生成二进制文件名字
$ GOOS=linux GOARCH=amd64 go build -o app.linux

# 编译到 linux 32bit
$ GOOS=linux GOARCH=386 go build

# 编译到 windows 64bit
$ GOOS=windows GOARCH=amd64 go build

# 编译到 windows 32bit
$ GOOS=windows GOARCH=386 go build

# 编译到 Mac OS X 64bit
$ GOOS=darwin GOARCH=amd64 go build

```


