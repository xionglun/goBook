#加密和解密数据
前面小节介绍了如何存储密码，但是有的时候，我们想把一些敏感数据加密后存储起来，在将来的某个时候，随需将它们解密出来，
此时我们应该在选用对称加密算法来满足我们的需求。

---------

### 对称加密
Go语言的`crypto`里面支持对称加密的高级加解密包有：
- `crypto/aes`包：AES(Advanced Encryption Standard)，又称Rijndael加密法，是美国联邦政府采用的一种区块加密标准。
- `crypto/des`包：DES(Data Encryption Standard)，是一种对称加密标准，是目前使用最广泛的密钥系统，
	特别是在保护金融数据的安全中。曾是美国联邦政府的加密标准，但现已被AES所替代。

因为这两种算法使用方法类似，所以在此，我们仅用aes包为例来讲解它们的使用，请看下面的例子
```go
package main

import (
	"crypto/aes"
	"crypto/cipher"
	"fmt"
	"os"
)

var commonIV = []byte{0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08, 0x09, 0x0a, 0x0b, 0x0c, 0x0d, 0x0e, 0x0f}

func main() {
	//需要去加密的字符串
	plaintext := []byte("My name is Astaxie")
	//如果传入加密串的话，plaint就是传入的字符串
	if len(os.Args) > 1 {
		plaintext = []byte(os.Args[1])
	}

	//aes的加密字符串
	key_text := "astaxie12798akljzmknm.ahkjkljl;k"
	if len(os.Args) > 2 {
		key_text = os.Args[2]
	}

	fmt.Println(len(key_text))

	// 创建加密算法aes
	c, err := aes.NewCipher([]byte(key_text))
	if err != nil {
		fmt.Printf("Error: NewCipher(%d bytes) = %s", len(key_text), err)
		os.Exit(-1)
	}

	//加密字符串
	cfb := cipher.NewCFBEncrypter(c, commonIV)
	ciphertext := make([]byte, len(plaintext))
	cfb.XORKeyStream(ciphertext, plaintext)
	fmt.Printf("%s=>%x\n", plaintext, ciphertext)

	// 解密字符串
	cfbdec := cipher.NewCFBDecrypter(c, commonIV)
	plaintextCopy := make([]byte, len(plaintext))
	cfbdec.XORKeyStream(plaintextCopy, ciphertext)
	fmt.Printf("%x=>%s\n", ciphertext, plaintextCopy)
}
```

上面通过调用函数`aes.NewCipher`(参数key必须是16、24或者32位的[]byte，分别对应AES-128, AES-192或AES-256算法),
返回了一个`cipher.Block`接口，这个接口实现了三个功能：
```go
	type Block interface {
		// BlockSize returns the cipher's block size.
		BlockSize() int

		// Encrypt encrypts the first block in src into dst.
		// Dst and src may point at the same memory.
		Encrypt(dst, src []byte)

		// Decrypt decrypts the first block in src into dst.
		// Dst and src may point at the same memory.
		Decrypt(dst, src []byte)
	}
```

这三个函数实现了加解密操作，详细的操作请看上面的例子。

---------

### 非对称加密
非对称加密使用最多的是**RSA**算法。非对称加密使用一套公钥与私钥的体系。

RSA加密：
```go
package main

import (
	"fmt"
	"crypto/rand"
	"crypto/rsa"
	"crypto/sha1"
	"crypto/x509"
	"encoding/pem"
)

func main() {

	msg := []byte("Content to be encrypted!")

	// 注意，本处是为了方便，直接将公私钥以字符串的形式给出了。
	// 获取公钥（公钥也可以从证书中读取）
	publicKeyData := `-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDZsfv1qscqYdy4vY+P4e3cAtmv
ppXQcRvrF1cB4drkv0haU24Y7m5qYtT52Kr539RdbKKdLAM6s20lWy7+5C0Dgacd
wYWd/7PeCELyEipZJL07Vro7Ate8Bfjya+wltGK9+XNUIHiumUKULW4KDx21+1NL
AUeJ6PeW+DAkmJWF6QIDAQAB
-----END PUBLIC KEY-----
`

	// 解码公钥
	pubBlock, _ := pem.Decode([]byte(publicKeyData))

	// 读取公钥
	pubKeyValue, err := x509.ParsePKIXPublicKey(pubBlock.Bytes)
	if err != nil {
		panic(err)
	}
	pub := pubKeyValue.(*rsa.PublicKey)

	// 加密数据方法1
	// 注意：用这个函数加密纯文本是很危险的，尽量使用下面的EncryptOAEP方法
	encryptPKCS15, err := rsa.EncryptPKCS1v15(rand.Reader, pub, msg)
	if err != nil {
		panic(err)
	}

	encryptOAEP, err := rsa.EncryptOAEP(sha1.New(), rand.Reader, pub, msg, nil)
	if err != nil {
		panic(err)
	}

	// 获取私钥
	privateKeyData := `-----BEGIN RSA PRIVATE KEY-----
MIICXQIBAAKBgQDZsfv1qscqYdy4vY+P4e3cAtmvppXQcRvrF1cB4drkv0haU24Y
7m5qYtT52Kr539RdbKKdLAM6s20lWy7+5C0DgacdwYWd/7PeCELyEipZJL07Vro7
Ate8Bfjya+wltGK9+XNUIHiumUKULW4KDx21+1NLAUeJ6PeW+DAkmJWF6QIDAQAB
AoGBAJlNxenTQj6OfCl9FMR2jlMJjtMrtQT9InQEE7m3m7bLHeC+MCJOhmNVBjaM
ZpthDORdxIZ6oCuOf6Z2+Dl35lntGFh5J7S34UP2BWzF1IyyQfySCNexGNHKT1G1
XKQtHmtc2gWWthEg+S6ciIyw2IGrrP2Rke81vYHExPrexf0hAkEA9Izb0MiYsMCB
/jemLJB0Lb3Y/B8xjGjQFFBQT7bmwBVjvZWZVpnMnXi9sWGdgUpxsCuAIROXjZ40
IRZ2C9EouwJBAOPjPvV8Sgw4vaseOqlJvSq/C/pIFx6RVznDGlc8bRg7SgTPpjHG
4G+M3mVgpCX1a/EU1mB+fhiJ2LAZ/pTtY6sCQGaW9NwIWu3DRIVGCSMm0mYh/3X9
DAcwLSJoctiODQ1Fq9rreDE5QfpJnaJdJfsIJNtX1F+L3YceeBXtW0Ynz2MCQBI8
9KP274Is5FkWkUFNKnuKUK4WKOuEXEO+LpR+vIhs7k6WQ8nGDd4/mujoJBr5mkrw
DPwqA3N5TMNDQVGv8gMCQQCaKGJgWYgvo3/milFfImbp+m7/Y3vCptarldXrYQWO
AQjxwc71ZGBFDITYvdgJM1MTqc8xQek1FXn1vfpy2c6O
-----END RSA PRIVATE KEY-----
`

	// 解析出私钥
	priBlock, _ := pem.Decode([]byte(privateKeyData))
	priKey, err := x509.ParsePKCS1PrivateKey(priBlock.Bytes)
	if err != nil {
		panic(err)
	}

	// 解密PKCS1v15加密的内容
	decryptPKCS, err := rsa.DecryptPKCS1v15(rand.Reader, priKey, encryptPKCS15)
	if err != nil {
		panic(err)
	}
	fmt.Println(string(decryptPKCS))

	// 解密RSA-OAEP方式加密后的内容
	decryptOAEP, err := rsa.DecryptOAEP(sha1.New(), rand.Reader, priKey, encryptOAEP, nil)
	if err != nil {
		panic(err)
	}
	fmt.Println(string(decryptOAEP))

}
```
