# Go 安装


### Linux 安装
安装步骤：
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

然后进行检测，执行`go`：

![](../images/1.1.mac.png?raw=true)

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

## Go源码安装
参见: [Golang源码安装](http://golang.org/doc/install/source)(需翻墙)


## Go标准包安装
Go提供了每个平台打好包的一键安装，这些包默认会安装到如下目录：/usr/local/go (Windows系统：c:\Go)；   

> 当然你可以改变他们的安装位置，但是改变之后你必须在你的环境变量中设置如下信息：
> ```sh
> export GOROOT=$HOME/go  
> export PATH=$PATH:$GOROOT/bin
> ```


### Windows 安装
访问[下载地址][downlink]，32位系统下载go1.4.windows-386.msi，64位系统下载go1.4.windows-amd64.msi。   
双击打开下载的文件，一路按照默认点击下一步，这个时候go已经安装到你的系统中。   
默认安装之后已经在你的系统环境变量中加入了`c:/go/bin`，这个时候打开cmd，输入`go`：

看到类似上面Linux安装成功的图片说明已经安装成功。


## 第三方工具安装
### GVM
gvm是第三方开发的Go多版本管理工具，类似ruby里面的rvm工具。使用起来相当的方便，安装gvm使用如下命令：
```sh
bash < <(curl -s https://raw.github.com/moovweb/gvm/master/binscripts/gvm-installer)
```

安装完成后我们就可以安装go了：
```
gvm install go1.5
gvm use go1.5
```
也可以使用下面的命令，省去每次调用`gvm use`的麻烦：
```
gvm use go1.5 --default
```

执行完上面的命令之后GOPATH、GOROOT等环境变量会自动设置好，这样就可以直接使用了。

### apt-get
Ubuntu是目前使用最多的Linux桌面系统，使用`apt-get`命令来管理软件包，我们可以通过下面的命令来安装Go，
为了以后方便，应该把 `git`也安装上：
```
sudo apt-get install python-software-properties
sudo add-apt-repository ppa:gophers/go
sudo apt-get update
sudo apt-get install golang-stable git-core
```

[downlink]: https://golang.org/dl/ "Go安装包下载"
