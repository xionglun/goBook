# 交叉编译
Go是一门编译型语言，所以在不同平台上，需要编译生成不同格式的二进制包。


### Mac上编译设置
1. 安装Go时候需要配置好交叉编译
  * 使用**Homebrew**安装并编译
    ```sh
    # common表示交叉编译的平台为linux, windows, freebsd这三个
    # 其它参数可以参考brew的命令
    $ brew install go --with-cc-common
    ```
    注意，下载时候可能需要翻墙。
  
  * 手工编译
    ```sh
    $ cd /usr/local/go/src
    $ sudo GOOS=windows GOARCH=386 CGO_ENABLED=0 ./make.bash --no-clean
    $ sudo GOOS=windows GOARCH=amd64 CGO_ENABLED=0 ./make.bash --no-clean
    $ sudo GOOS=linux GOARCH=386 CGO_ENABLED=0 ./make.bash --no-clean
    $ sudo GOOS=linux GOARCH=amd64 CGO_ENABLED=0 ./make.bash --no-clean
    $ sudo GOOS=freebsd GOARCH=386 CGO_ENABLED=0 ./make.bash --no-clean
    $ sudo GOOS=freebsd GOARCH=amd64 CGO_ENABLED=0 ./make.bash --no-clean
    ```

2. 进行编译   
当安装完成之后，就可以进行交叉编译了。示例：
  ```sh
  # 编译到 linux 64bit
  $ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build
  或者可以使用 -o 选项指定生成二进制文件名字
  $ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o app.linux

  # 编译到 linux 32bit
  $ CGO_ENABLED=0 GOOS=linux GOARCH=386 go build

  # 编译到 windows 64bit
  $ CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build

  # 编译到 windows 32bit
  $ CGO_ENABLED=0 GOOS=windows GOARCH=386 go build
  ```


