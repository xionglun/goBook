# Go 安装


### Linux 安装
(需翻墙)安装步骤：
```sh
# 下载go安装包，按照系统版本和Go版本更改链接
$ wget https://storage.googleapis.com/golang/go1.5.linux-amd64.tar.gz
# 解压到/usr/local/go这里
$ sudo tar -C /usr/local -xzf go1.5.linux-amd64.tar.gz
# 修改PATH环境变量，并设置GOPATH，其中GOPATH可以使用任意自定义位置（工作目录）。
$ vim ~/.profile  # 编辑 ~/.bashrc
  > export PATH=$PATH:/usr/local/go/bin
  > export GOPATH=$HOME/go
```

> 如不能翻墙，可以将下载地址替换为：
> http://allen.qiniudn.com/go1.5.linux-amd64.tar.gz

然后进行检测，执行`go`：

![](../images/1.1.linux.png?raw=true)

图1.2 Linux系统下安装成功之后执行go显示的信息

如果出现go的Usage信息，那么说明go已经安装成功了；   
如果出现该命令不存在，那么可以检查一下自己的PATH环境变中是否包含了go的安装目录。

### Mac 安装
访问[下载地址][downlink](需翻墙)。   
32位系统下载go1.5.darwin-386-osx10.X.pkg，64位系统下载go1.5.darwin-amd64-osx10.X.pkg。
双击下载文件，一路默认安装点击下一步，这个时候go已经安装到你的系统中。
默认已经在PATH中增加了相应的`~/go/bin`,这个时候打开终端，输入`go`。   
如果出现go的Usage信息，那么说明go已经安装成功了；   
如果出现该命令不存在，那么可以检查一下自己的PATH环境变中是否包含了go的安装目录。   

#### 使用Homebrew安装：   
这里需要先安装[homebrew](http://brew.sh/index_zh-cn.html)。然后使用下面命令即可一键安装或更新:   
```sh
 // 只安装go
 $ brew install go
 // 带跨平台编译功能安装
 $ brew install go --with-cc-common 
 
 // 更新go
 $ brew upgrade go
```

### Go源码安装
参见: [Golang源码安装](http://golang.org/doc/install/source)(需翻墙)


### Windows 安装
按照系统对应下面下载地址下载：   
 * 32位系统下载 go1.5.windows-386.msi
 * 64位系统下载 go1.5.windows-amd64.msi

双击打开下载的文件，一路按照默认点击下一步，这个时候go已经安装到你的系统中。   
默认安装之后已经在你的系统环境变量中加入了`c:/go/bin`，这个时候打开cmd，输入`go`：

看到类似上面Linux安装成功的图片说明已经安装成功。


### ☟无需翻墙链接
Go 1.5下载地址：   
* [go1.5.src.tar.gz](http://allen.qiniudn.com/go1.5.src.tar.gz)
* [go1.5.darwin-amd64.tar.gz](http://allen.qiniudn.com/go1.5.darwin-amd64.tar.gz)
* [go1.5.darwin-amd64.pkg](http://allen.qiniudn.com/go1.5.darwin-amd64.pkg)
* [go1.5.linux-amd64.tar.gz](http://allen.qiniudn.com/go1.5.linux-amd64.tar.gz)
* [go1.5.windows-386.msi](http://allen.qiniudn.com/go1.5.windows-386.msi)
* [go1.5.windows-amd64.msi](http://allen.qiniudn.com/go1.5.windows-amd64.msi)
* [go1.5.windows-amd64.zip](http://allen.qiniudn.com/go1.5.windows-amd64.zip)


[downlink]: https://golang.org/dl/ "Go安装包下载"
