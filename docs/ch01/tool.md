# Go开发工具
本节我将介绍几个开发工具，它们都具有自动化提示，自动化fmt功能。因为它们都是跨平台的，所以安装步骤之类的都是通用的。

### Vim
Vim是从vi发展出来的一个文本编辑器, 代码补全、编译及错误跳转等方便编程的功能特别丰富，在程序员中被广泛使用。

 1. 安装[Vundle](https://github.com/VundleVim/Vundle.vim) 
  ```
  $ git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
  ```

 2. 在~/.vimrc文件中增加golang插件`fatih/vim-go`
  ```
  Plugin 'fatih/vim-go'
  ```

 3. 安装
  ```
  :PluginInstall
  ```

### Sublime Text
 
   这里将介绍Sublime Text 2（以下简称Sublime）+GoSublime+gocode+MarGo的组合，那么为什么选择这个组合呢？
 
   - 自动化提示代码,如下图所示
 
 	![](../images/1.4.sublime1.png?raw=true)
 
 	图1.5 sublime自动化提示界面
 
   - 保存的时候自动格式化代码，让您编写的代码更加美观，符合Go的标准。
   - 支持项目管理
 	
 	![](../images/1.4.sublime2.png?raw=true)
 	
 	图1.6 sublime项目管理界面
 	
   - 支持语法高亮
   - Sublime Text 2可免费使用，只是保存次数达到一定数量之后就会提示是否购买，点击取消继续用，和正式注册版本没有任何区别。
 
 
 接下来就开始讲如何安装，下载[Sublime](http://www.sublimetext.com/)
 
   根据自己相应的系统下载相应的版本，然后打开Sublime，对于不熟悉Sublime的同学可以先看一下这篇文章[Sublime Text 2 入门及技巧](http://lucifr.com/139225/sublime-text-2-tricks-and-tips/)
 
   1. 打开之后安装 Package Control：Ctrl+` 打开命令行，执行如下代码：
    ```
    import urllib2,os; pf='Package Control.sublime-package'; ipp=sublime.installed_packages_path(); os.makedirs(ipp) if not os.path.exists(ipp) else None; urllib2.install_opener(urllib2.build_opener(urllib2.ProxyHandler())); open(os.path.join(ipp,pf),'wb').write(urllib2.urlopen('http://sublime.wbond.net/'+pf.replace(' ','%20')).read()); print 'Please restart Sublime Text to finish installation'
    ```
    这个时候重启一下Sublime，可以发现在在菜单栏多了一个如下的栏目，说明Package Control已经安装成功了。
 
   ![](../images/1.4.sublime3.png?raw=true)
   图1.7 sublime包管理
 
   2. 安装完之后就可以安装Sublime的插件了。
      需安装GoSublime、SidebarEnhancements和Go Build，安装插件之后记得重启Sublime生效，
      Ctrl+Shift+p打开Package Controll 输入`pcip`（即“Package Control: Install Package”的缩写）。   
      这个时候看左下角显示正在读取包数据，完成之后出现如下界面
      ![](../images/1.4.sublime4.png?raw=true)
      图1.8 sublime安装插件界面   
      这个时候输入GoSublime，按确定就开始安装了。同理应用于SidebarEnhancements和Go Build。
 
   3. 验证是否安装成功，你可以打开Sublime，打开main.go，看看语法是不是高亮了，
      输入`import`是不是自动化提示了，`import "fmt"`之后，输入`fmt.`是不是自动化提示有函数了。   
      如果已经出现这个提示，那说明你已经安装完成了，并且完成了自动提示。   
      如果没有出现这样的提示，一般就是你的`$PATH`没有配置正确。
      你可以打开终端，输入gocode，是不是能够正确运行，如果不行就说明`$PATH`没有配置正确。
      (针对XP)有时候在终端能运行成功,但sublime无提示或者编译解码错误,请安装sublime text3和convert utf8插件试一试
 
   4. MacOS下已经设置了$GOROOT, $GOPATH, $GOBIN，还是没有自动提示怎么办。   
      请在sublime中使用command + 9， 然后输入env检查$PATH, GOROOT, $GOPATH, $GOBIN等变量，
      如果没有请采用下面的方法。   
      首先建立下面的连接， 然后从Terminal中直接启动sublime
      ```
      ln -s /Applications/Sublime\ Text\ 2.app/Contents/SharedSupport/bin/subl /usr/local/bin/sublime
      ```
   5. 项目支持,让sublime支持项目本身的pkg库提示,有两种基本的实现   
      一种为设定 gosublime 插件的 `Setting - user` 配置 
      ```json
        {
          "env": { "GOPATH": "$HOME/golang:$GS_GOPATH" }
        }
      ```
      `$GS_GOPATH` 是 gosublime 的伪环境变量, 它自动寻找 `.go`文件所在的 `~/go/src` 来推测 `~/go/` 为项目位置, 
      从而自动适应 `GOPATH` 。在这里你应当将$HOME/golang换成你自己的go目录路径。
      （注意：使用这种方式会在sublime内覆盖原有的GOPATH，如果这里设置出错，会产生GOPATH相关的问题）
 
   另外一种为保存sublime 项目 , 修改 project_name.sublime-project 添加节点
   ```
 		"settings": {
 			"GoSublime": {
 				"env": {
 					"GOPATH": "$HOME/golang/pwd" //此处修改为项目路径
 				}
 			}
 		},
 
 		"folders"{...
   ```


## LiteIDE

  LiteIDE是一款专门为Go语言开发的跨平台轻量级集成开发环境（IDE），由visualfc编写。

  ![](../images/1.4.liteide.png?raw=true)

图1.4 LiteIDE主界面

**LiteIDE主要特点：**

* 支持主流操作系统
	* Windows 
	* Linux 
	* MacOS X
* Go编译环境管理和切换
	* 管理和切换多个Go编译环境
	* 支持Go语言交叉编译
* 与Go标准一致的项目管理方式
	* 基于GOPATH的包浏览器
	* 基于GOPATH的编译系统
	* 基于GOPATH的Api文档检索
* Go语言的编辑支持
	* 类浏览器和大纲显示
	* Gocode(代码自动完成工具)的完美支持
	* Go语言文档查看和Api快速检索
	* 代码表达式信息显示`F1`
	* 源代码定义跳转支持`F2`
	* Gdb断点和调试支持
	* gofmt自动格式化支持
* 其他特征
	* 支持多国语言界面显示
	* 完全插件体系结构
	* 支持编辑器配色方案
	* 基于Kate的语法显示支持
	* 基于全文的单词自动完成
	* 支持键盘快捷键绑定方案
	* Markdown文档编辑支持
		* 实时预览和同步显示
		* 自定义CSS显示
		* 可导出HTML和PDF文档
		* 批量转换/合并为HTML/PDF文档

**LiteIDE安装配置**

* LiteIDE安装
	* 下载地址 <http://sourceforge.net/projects/liteide/files>
	* 源码地址 <https://github.com/visualfc/liteide>
	
	首先安装好Go语言环境，然后根据操作系统下载LiteIDE对应的压缩文件直接解压即可使用。

* 安装Gocode

	启用Go语言的输入自动完成需要安装Gocode：
	```
	go get -u github.com/nsf/gocode
  ```
* 编译环境设置

	根据自身系统要求切换和配置LiteIDE当前使用的环境变量。
	
	以Windows操作系统，64位Go语言为例，
	工具栏的环境配置中选择win64，点`编辑环境`，进入LiteIDE编辑win64.env文件
	```
	GOROOT=c:\go
	GOBIN=
	GOARCH=amd64
	GOOS=windows
	CGO_ENABLED=1
		
	PATH=%GOBIN%;%GOROOT%\bin;%PATH%
	...
	```
	将其中的`GOROOT=c:\go`修改为当前Go安装路径，保存即可，如果有MinGW64，可以将`c:\MinGW64\bin`加入PATH中以便go调用gcc支持CGO编译。

	以Linux操作系统，64位Go语言为例，
	工具栏的环境配置中选择linux64，点`编辑环境`，进入LiteIDE编辑linux64.env文件
	```
	GOROOT=$HOME/go
	GOBIN=
	GOARCH=amd64
	GOOS=linux
	CGO_ENABLED=1
	
	PATH=$GOBIN:$GOROOT/bin:$PATH	
	...
	```
	将其中的`GOROOT=$HOME/go`修改为当前Go安装路径，存盘即可。

* GOPATH设置

	Go语言的工具链使用GOPATH设置，是Go语言开发的项目路径列表，在命令行中输入(在LiteIDE中也可以`Ctrl+,`直接输入)`go help gopath`快速查看GOPATH文档。
	
	在LiteIDE中可以方便的查看和设置GOPATH。通过`菜单－查看－GOPATH`设置，可以查看系统中已存在的GOPATH列表，
	同时可根据需要添加项目目录到自定义GOPATH列表中。

