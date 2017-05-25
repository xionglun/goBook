# Go开发工具
本节我将介绍几个开发工具，它们都具有自动化提示，自动化fmt功能。因为它们都是跨平台的，所以安装步骤之类的都是通用的。

### VS Code
[Visual Studio Code](https://code.visualstudio.com)是微软基于Electron推出的一款开源轻量级文本编辑器，
其与[Atom](https://atom.io)类似，但是速度更快，编码更顺手，非常推荐使用。


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

### 其它
- [Goglang](https://www.jetbrains.com/go/) 是Jetbrains推出的一款Golang IDE，其外观用法与IDEA基本一致。
- [Spacemacs](https://github.com/syl20bnr/spacemacs) 一个基于Emacs但具有VIM风格的文本编辑器。
- [Sublime Text](http://www.sublimetext.com/) 一款非常流行轻量级文本编辑器。
