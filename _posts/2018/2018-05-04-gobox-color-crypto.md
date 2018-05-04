---
layout: post
title:  "gobox中的color和crypto"
---

今天来说下gobox中的color和crypto两个box。

## color

color的主要作用是为在终端中输出的信息添加颜色。

### 用法示例

```
package main

import (
	"github.com/goinbox/color"

	"fmt"
)

func main() {
	fmt.Println(string(color.Black([]byte("Black"))))
	fmt.Println(string(color.Red([]byte("Red"))))
	fmt.Println(string(color.Green([]byte("Green"))))
	fmt.Println(string(color.Yellow([]byte("Yellow"))))
	fmt.Println(string(color.Blue([]byte("Blue"))))
	fmt.Println(string(color.Maganta([]byte("Maganta"))))
	fmt.Println(string(color.Cyan([]byte("Cyan"))))
	fmt.Println(string(color.White([]byte("White"))))
}
```

### 输出效果示例

![颜色效果](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-05-04-gobox-color-crypto/color.png?raw=true)

### 特别说明

如果用户想自定义颜色，可以调用如下方法：

```
func Color(color []byte, msg []byte) []byte
```

其中color为命令行颜色编码，可以参考：https://www.ibm.com/developerworks/cn/linux/l-tip-prompt/tip01/

## crypto

crypto提供了常用的加解密方法。

### md5

方法：

```
func Md5String(data []byte) string
```

使用示例：

```
package main

import (
	"github.com/goinbox/crypto"

	"fmt"
)

func main() {
	mstr := crypto.Md5String([]byte("test md5"))

	fmt.Println(mstr)
}
```

程序输出：

```
0e4e3b2681e8931c067a23c583c878d5
```

### AES对称加解密

先看一个示例：

```
package main

import (
	"github.com/goinbox/crypto"

	"fmt"
)

func main() {
	key := crypto.Md5String([]byte("gobox-crypto"))
	iv := crypto.Md5String([]byte("aes"))[:crypto.AES_BLOCK_SIZE]
	data := []byte("abcdefg")

	acc, _ := crypto.NewAesCBCCrypter([]byte(key), []byte(iv))
	crypted := acc.Encrypt(data)
	fmt.Println("crypted:", crypted)

	dcrypted := acc.Decrypt(crypted)
	fmt.Println("dcrypted:", string(dcrypted))
}
```

输出：

```
crypted: [22 146 17 181 62 224 155 54 244 184 255 116 134 113 40 170]
dcrypted: abcdefg
```

大家可能对里面的参数不太理解，请看下这篇文章：http://blog.7rule.com/2018/03/09/crypto-base64.html，里面讲对称加密的部分会解答你的疑问。

#### 特别注意

默认的加解密填充方式，使用的`PKCS5方式`，如果需要使用自己的填充方式，请实现接口：

```
type PaddingInterface interface {
	Padding(data []byte) []byte
	UnPadding(data []byte) []byte
}
```

之后在加解密前调用：

```
func (a *AesCBCCrypter) SetPadding(padding PaddingInterface)
```

欢迎大家使用，使用中有遇到问题随时反馈，我们会尽快响应，谢谢！
