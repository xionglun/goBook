# Hash散列
Hash散列(又称哈希)是一种从任何一种数据中创建小的数字“指纹”的方法。
散列函数把消息或数据压缩成摘要，使得数据量变小，将数据的格式固定下来。
该函数将数据打乱混合，重新创建一个叫做散列值（hash values，hash codes，hash sums，或hashes）的指纹。
散列值通常用来代表一个短的随机字母和数字组成的字符串。

### BASE64
**BASE64** 其实不是一种散列算法，更不是一种加密方法。经过base64编码后可以解码还原到原字符串。
> Base64是一种基于64个可打印字符来表示二进制数据的表示方法。
> Base64常用于在通常处理文本数据的场合，表示、传输、存储一些二进制数据。包括MIME的email、在XML中存储复杂数据。

其简单使用方法如下:
```go
package main

import (
	"encoding/base64"
	"fmt"
)

func base64Encode(src []byte) []byte {
	return []byte(base64.StdEncoding.EncodeToString(src))
}

func base64Decode(src []byte) ([]byte, error) {
	return base64.StdEncoding.DecodeString(string(src))
}

func main() {
	// encode
	hello := "你好，世界！ hello world"
	debyte := base64Encode([]byte(hello))
	fmt.Println(debyte)
	// decode
	enbyte, err := base64Decode(debyte)
	if err != nil {
		fmt.Println(err.Error())
	}

	if hello != string(enbyte) {
		fmt.Println("hello is not equal to enbyte")
	}
	fmt.Println(string(enbyte))
}
```

注意，除了`base64.StdEncoding`之外，常用的还有`base64.URLEncoding`使目标转换为URL可用的字符串。

### MD5
**MD5** 消息摘要算法是一种被广泛使用的密码散列函数，可以产生出一个128位（16字节）的散列值，用于确保信息传输完整一致。
> MD5算法在1996年后被证实存在弱点，可以被加以破解。对于需要高度安全性的数据，一般建议改用其他算法。
> 因其普遍、稳定、快速的特点，仍广泛应用于普通数据的错误检查领域。如检验下载到的文件完整性。

md5使用示例:
```go
package main

import (
	"crypto/md5"
	"fmt"
)

func main() {
	data := []byte("These pretzels are making me thirsty.")
	fmt.Printf("%x", md5.Sum(data))
}
```

### sha1
**sha1** 是SHA家族的五个算法之一（其它四个是SHA-224、SHA-256、SHA-384，和SHA-512）。

sha1使用示例：
```go
package main

import (
    "crypto/sha1"
    "fmt"
)

func main() {
    s := "this is a string"
    h := sha1.New()
    h.Write([]byte(s))
    sum := h.Sum(nil)

    fmt.Println("%x\n", sum)
}
```

其它SHA算法使用与上大同小异。
