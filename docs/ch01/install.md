# Go 安装
Golang 安装有多种方式，详见[官网](https://golang.org/doc/install)

### Linux/Unix/macOS 安装
此方式为使用已编译二进制压缩包安装64位Go

安装步骤：   
1. 下载go安装包(需翻墙，墙内地址见下方引用)    
  ```sh
  # 按照系统版本和Go版本更改链接
  $ wget https://storage.googleapis.com/golang/go1.8.3.linux-amd64.tar.gz
  ```

2. 解压到/usr/local
  ```sh
  $ sudo tar -C /usr/local -xzf go1.8.3.linux-amd64.tar.gz
  ```

3. 修改PATH环境变量
  ```sh
  $ vim ~/.profile  # 或者 ~/.bashrc
  # 添加: export PATH=$PATH:/usr/local/go/bin
  ```

4. (可选)设置GOPATH
  ```sh
  # go1.8及后续版本在没有设置GOPATH时，会默认以$HOME/go为GOPATH
  # GOPATH为你的工作目录，其可以为任意合法目录
  $ vim ~/.profile  # 或者 ~/.bashrc
  # 添加: export GOPATH=$HOME/go
  ```

然后进行检测，执行`go`：

![](../images/1.1.linux.png?raw=true)

图1.2 Linux系统下安装成功之后执行go显示的信息

如果出现go的Usage信息，那么说明go已经安装成功了；   
如果出现该命令不存在，那么可以检查一下自己的**PATH**环境变中是否包含了go的安装目录。

### macOS 安装包安装
访问[下载地址][downlink](需翻墙)。

下载(需翻墙): [go1.8.3.darwin-amd64.pkg](https://storage.googleapis.com/golang/go1.8.3.darwin-amd64.pkg)

双击下载文件，一路默认安装点击下一步即可。   

### macOS 使用Homebrew安装：   
这里需要先安装[homebrew](http://brew.sh/index_zh-cn.html)。然后使用下面命令即可一键安装或更新:   
```sh
 // 安装go
 $ brew install go
 
 // 更新go
 $ brew upgrade go
```

### Go源码安装
参见: [Golang源码安装](http://golang.org/doc/install/source)(需翻墙)


### Windows 安装
按照系统对应下面下载地址下载：   
 * 32位系统下载 go1.8.3.windows-386.msi
 * 64位系统下载 go1.8.3.windows-amd64.msi

双击打开下载的文件，一路按照默认点击下一步即可。


[downlink]: https://golang.org/dl/ "Go安装包下载"
