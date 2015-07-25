#文件操作
在任何计算机设备中，文件是都是必须的对象，而在Web编程中,文件的操作一直是Web程序员经常遇到的问题,
文件操作在Web应用中是必须的,非常有用的,我们经常遇到生成文件目录,文件(夹)编辑等操作,
现在我把Go中的这些操作做一详细总结并实例示范如何使用。


## 目录操作
文件操作的大多数函数都是在`os`包里面，下面列举了几个目录操作的：

- `func Mkdir(name string, perm FileMode) error`   
	创建名称为name的目录，权限设置是perm，例如0777   
- `func MkdirAll(path string, perm FileMode) error`   
	根据path创建多级子目录，例如astaxie/test1/test2。   
- `func Remove(name string) error`   
	删除名称为name的目录，当目录下有文件或者其他目录是会出错   
- `func RemoveAll(path string) error`   
	根据path删除多级子目录，如果path是单个名称，那么该目录不删除。   
- `func Readdir(n int) (fi []FileInfo, err error)`   
    读取一个目录中文件列表   
	- 如果n > 0，则至多读取并返回n个文件信息   
	- 如果n <= 0，则读取所有的文件列表信息

下面是演示代码：
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	rootPath := "root"
	err := os.Mkdir(rootPath, 0777)
	if err != nil {
		fmt.Println("mkdir root failed!")
		return
	}
	rrPath := "root/test1/test2"
	err := os.MkdirAll(rrPath, 0777)
	if err != nil {
		fmt.Println("mkdir root/test1/test2 failed!")
		return
	}
	err := os.Remove(rootPath)
	if err != nil {
		fmt.Println(err)
	}
	err := os.RemoveAll(rootPath)
	if err != nil {
		fmt.Println(err)
	}	
}
```

## 文件操作
### 新建与打开文件
新建文件可以通过如下两个方法:   
- `func Create(name string) (file *File, err Error)`   
	根据提供的文件名创建新的文件，返回一个文件对象，默认权限是0666的文件，返回的文件对象是可读写的。   
- `func NewFile(fd uintptr, name string) *File`   
	根据文件描述符创建相应的文件，返回一个文件对象   

通过如下两个方法来打开文件：   
- `func Open(name string) (file *File, err Error)`    
	该方法以只读方式打开一个文件，内部实现其实调用了OpenFile。    
- `func OpenFile(name string, flag int, perm uint32) (file *File, err Error)`   	
	打开某一个文件，flag是打开的方式，只读、读写等，perm是权限		
	```
	// flag 可取值:
	O_RDONLY   // open the file read-only 以只读方式打开
	O_WRONLY   // open the file write-only 以只写的方式打开
	O_RDWR     // open the file read-write 以读写的方式打开
	O_APPEND   // append data to the file  添加内容到文件，不能单独使用，需要在可写条件下
	O_CREATE   // create a new file if none exists 如果文件不存在，则创建一个新文件
	O_EXCL     // used with O_CREATE, file must not exist 和标志O_CREATE一起用，文件必须是不存在的
	O_SYNC     // open for synchronous I/O 以同步I/O的方式打开
	O_TRUNC    // if possible, truncate file when opened 在条件允许的情况下，打开文件时候清空文件内容
	```

### 写文件
写文件函数：   
- `func (file *File) Write(b []byte) (n int, err Error)`   
	写入byte数组类型的信息到文件   
- `func (file *File) WriteAt(b []byte, off int64) (n int, err Error)`   
	在指定偏移位置off处开始写入byte数组类型的信息   
- `func (file *File) WriteString(s string) (ret int, err Error)`   
	写入字符串s到文件   
	
写文件的示例代码
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	userFile := "test.txt"
	// 以只写且添加的方式打开文件，文件权限为0666
	fout, err := os.OpenFile(userFile, os.O_WRONLY|os.O_APPEND, 0666)
	if err != nil {
		fmt.Println(userFile, err)
		return
	}
	defer fout.Close()
	for i := 0; i < 10; i++ {
		fout.WriteString("Just a test!\r\n")
		fout.Write([]byte("Just a test!\r\n"))
	}
}
```

### 读文件
读文件函数：   
- `func (file *File) Read(b []byte) (n int, err Error)`   
	读取数据到b中   
- `func (file *File) ReadAt(b []byte, off int64) (n int, err Error)`   
	从off开始读取数据到b中   

读文件的示例代码:
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	userFile := "test.txt"
	fl, err := os.Open(userFile)
	if err != nil {
		fmt.Println(userFile, err)
		return
	}
	defer fl.Close()
	buf := make([]byte, 1024)
	for {
		n, _ := fl.Read(buf)
		if 0 == n {
			break
		}
		os.Stdout.Write(buf[:n])
	}
}
```

### 删除文件
Go语言里面删除文件和删除文件夹是同一个函数   
- `func Remove(name string) Error`   
	调用该函数就可以删除文件名为name的文件


## 示例
读取一个目录，并递归列出所有文件
```go
package main

import (
  "fmt"
  "log"
  "os"
  "path"
  "strings"
)

func printFile(file os.FileInfo, root string) {
  // 文件全路径
  filename := path.Join(root, file.Name())
  fmt.Println(filename)
}

func printDir(dir []os.FileInfo, root string, deep int) {
  // 如果深度超过3，则返回
  // 此处仅为测试时防止控制台打印输出太多
  if deep > 3 {
    return
  }
  for _, v := range dir {
    name := v.Name()
	// 忽略.和..
    if strings.HasPrefix(name, ".") {
      continue
    }
    isDir := v.IsDir()
    if !isDir {
      printFile(v, root)
    } else {
      _path := path.Join(root, name)
	  // 打印目录名称
      fmt.Println(_path)
      _dirpath, err := os.Open(_path)
      if err != nil {
        log.Fatal(err)
      }
      defer _dirpath.Close()
      _dir, err := _dirpath.Readdir(0)
      if err != nil {
        log.Fatal(err)
      }
	  // 递归目录
      printDir(_dir, _path, deep+1)
    }
  }
}

func main() {
  // 读取go目录, just for example!
  rootPath := "/usr/local/go"
  rootDir, err := os.Open(rootPath)
  if err != nil {
    log.Fatal(err)
  }
  defer rootDir.Close()

  fs, err := rootDir.Readdir(0)
  printDir(fs, rootPath, 0)

}
```

