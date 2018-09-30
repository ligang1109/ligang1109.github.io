---
layout: post
title:  "应用编程基础课第三讲：Go编程基础"
---

上面两次课我讲解了编程方面的基础知识，这次开始，我使用Go语言来做一些编程实践方面的讲解。

今天先来说下Go语言中的一些我认为比较重要的知识点。

关于Go的基础使用，这里不做过多介绍，可以阅读：

1. How to Write Go Code：https://golang.org/doc/code.html
1. Effective Go：https://golang.org/doc/effective_go.html
1. The Way to Go：https://github.com/Unknwon/the-way-to-go_ZH_CN

# 重要的数据结构

## slice

### 基础知识

slice是go中最常用的数据结构之一，它相当于动态数组，了解下它的内部实现，对我们是用来说有很大的好处：

slice的数据结构示例为：

```
type slice struct {
    ptr *array  //底层存储数组
    len int     //当前存储了多少个元素
    cap int     //底层数组可以存储多少个元素（从ptr指向的位置开始）
}
```

用张图来表示：

![slice-struct](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-30/go-slices-usage-and-internals_slice-struct.png?raw=true)

我们常用的slice有个len和cap的概念，他们就是取len和cap这两个字段的值。

slice我们通常都用它做为动态数组使用，但slice翻译过来是切片的意思，为什么呢？

我们来看个例子：

首先，我们创建一个slice：

```
s := make([]int, 5)
```

对应的数据结构为：

![slice-1](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-30/go-slices-usage-and-internals_slice-1.png?raw=true)

之后，我们再调用：

```
ss := s[2:4]
```

我们得到：

![slice-2](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-30/go-slices-usage-and-internals_slice-2.png?raw=true)

所以两个slice，相当于是在底层array上的两个切片。大家请注意下第二个slice的cap是3。

### 使用注意

slice在使用中有几个很容易出错的地方，需要大家注意下。

这里先总结下最容易出错的原因，就是多个slice在使用同样的底层存储时，修改一个slice会导致其它slice中的数据变化。

示例1：
```
s := []int{1, 2, 3}
fmt.Println(s)

ss := s[1:3]
ss[0] = 0
fmt.Println(s, ss)

s[1] = 11
fmt.Println(s, ss)
```

输出：
```
[1 2 3]
[1 0 3] [0 3]
[1 11 3] [11 3]
```

大家可以看到，由于两个slice都是用同样的底层array，所以修改其中一个就会导致另外一个的变化

示例2：
```
func main() {
    s := []int{1, 2, 3}
    fmt.Println(s)

    foo(s) or foo(s[1:3])
    fmt.Println(s)
}

func foo(ss []int) {
    ss[0] = 0
}
```

输出：
```
[1 2 3]
[1 0 3]
```

这个和上面同样的原因

示例3：
```
s := []int{1, 2, 3}
fmt.Println(s)

ss := s[1:3]
ss = append(ss, 4)
fmt.Println(s, ss)
```

输出：
```
[1 2 3]
[1 2 3] [2 3 4]
```

这里大家可以看到，由于append操作改变了其中一个slice的底层array，所以对其中一个slice的修改不会影响到另外一个。

## map

关于map，有如下几个地方需要注意：

- 使用先要初始化

```
var m map[string]int

m["a"] = 1
```

会导致：
```
panic: assignment to entry in nil map
```

正确使用：

```
m := make(map[string]int)
m["a"] = 1 

fmt.Println(m)
```

输出：
```
map[a:1]
```

- map作为函数形参时，函数中对map的修改会影响实参中的值

```
func main() {
    m := make(map[string]int)
    m["a"] = 1
    fmt.Println(m)

    foo(m)
    fmt.Println(m)
}   

func foo(fm map[string]int) {
    fm["a"] = 11
}
```

输出：
```
map[a:1]
map[a:11]
```

- 对map做并发读写会导致panic

```
var gm map[int]int

func main() {
    gm = make(map[int]int)

    for i := 0; i < 10; i++ {
        go foo(i)
    }

    time.Sleep(time.Second * 10)
}

func foo(i int) {
    for j := 0; j < 100; j++ {
        gm[i] = j
    }
}
```

运行结果：
```
fatal error: concurrent map writes
fatal error: concurrent map writes

goroutine 17 [running]:
runtime.throw(0x46ff50, 0x15)
	/usr/local/go/src/runtime/panic.go:616 +0x81 fp=0xc420028758 sp=0xc420028738 pc=0x422711
runtime.mapassign_fast64(0x45e4e0, 0xc42007a060, 0x0, 0x0)
	/usr/local/go/src/runtime/hashmap_fast.go:531 +0x2f6 fp=0xc4200287a0 sp=0xc420028758 pc=0x408306
main.foo(0x0)
	/home/ligang/tmp/go/main.go:22 +0x4c fp=0xc4200287d8 sp=0xc4200287a0 pc=0x44f4dc
runtime.goexit()
	/usr/local/go/src/runtime/asm_amd64.s:2361 +0x1 fp=0xc4200287e0 sp=0xc4200287d8 pc=0x448a51
created by main.main
	/home/ligang/tmp/go/main.go:14 +0x61
```

所以对map做并发读写时需要加锁

# 类型转换

我们开发强类型语言程序时通常需要做类型转换，Go中的类型转换有两种最常用的形式：

## 原生类型转换

- 同一大类型下（如整数的int、int64，浮点数的float32、float64等），可以用类型加括号的形式，如：

int -> int64：

```
var a int = 1
b := int64(a)
```

- 不同大类型下的转换，使用strconv包中的方法

## 复杂类型转换，通常是interface转指定类型

这个要使用类型断言：

```
var a interface{} = 1
b := a.(int)
```

请注意这里如果类型断言失败的话，程序会panic，可以使用recover防止：

```
defer func() {
	if r := recover(); r != nil {
		fmt.Println(r)
	}   
}()

var a interface{} = 1 
b := a.(string)
```

输出：
```
interface conversion: interface {} is int, not string
```

# 函数传参时的指针和结构体

这里只需要记住一点，就是结构体作为函数形参时，会做值拷贝，所以拷贝的那部分值的修改，不会反映到实参值

```
type ta struct { 
    i int
}

func main() { 
    var a ta
    a.i = 1 
    foo(a)

    fmt.Println(a)
}

func foo(t ta) { 
    t.i = 11
}
```

输出：

```
{1}
```

同样的：

```
type ta struct { 
    i int
}

func main() { 
    var a ta
    a.i = 1 
    a.foo()
    
    fmt.Println(a)
}

func (t ta) foo() {
    t.i = 11
}
```

输出：

```
{1}
```

指针就不同了，会修改实参中的原值，这里就不举例了。

# 防止栈溢出，递归转循环

我们编程时有时会写递归函数，递归虽然简单，但是会有栈溢出的风险，解决方法是把递归转循环，将存储从栈空间转移到堆空间上。

我们这里举个实际的例子，linux中有个`tree`命令，它能列出一个给定根目录下所有的文件，包括子目录：

```
ligang@vm-xubuntu ~/devspace/hogwarts $ tree cppsimple/
cppsimple/
├── cmake-build-debug
│   ├── CMakeCache.txt
│   ├── CMakeFiles
│   │   ├── 3.12.2
│   │   │   ├── CMakeCCompiler.cmake
│   │   │   ├── CMakeCXXCompiler.cmake
│   │   │   ├── CMakeDetermineCompilerABI_C.bin
│   │   │   ├── CMakeDetermineCompilerABI_CXX.bin
│   │   │   ├── CMakeSystem.cmake
│   │   │   ├── CompilerIdC
│   │   │   │   ├── a.out
│   │   │   │   ├── CMakeCCompilerId.c
│   │   │   │   └── tmp
│   │   │   └── CompilerIdCXX
│   │   │       ├── a.out
│   │   │       ├── CMakeCXXCompilerId.cpp

```

读取目录下的包括子目录的所有文件，最先想到的就是递归了，但是如果目录层级过深，显然会导致栈溢出，所以这是一个非常好的例子

实现代码如下：

```
func ListFilesInDir(rootDir string) ([]string, error) {
	rootDir = strings.TrimRight(rootDir, "/")
	if !DirExist(rootDir) {
		return nil, errors.New("Dir not exists")
	}

	var fileList []string
	dirList := []string{rootDir}

	for i := 0; i < len(dirList); i++ {
		curDir := dirList[i]
		file, err := os.Open(dirList[i])
		if err != nil {
			return nil, err
		}

		fis, err := file.Readdir(-1)
		if err != nil {
			return nil, err
		}

		for _, fi := range fis {
			path := curDir + "/" + fi.Name()
			if fi.IsDir() {
				dirList = append(dirList, path)
			} else {
				fileList = append(fileList, path)
			}
		}
	}

	return fileList, nil
}
```

由于slice这种动态存储结构使用的是在堆上的空间，所以我们将递归转循环解决这个问题。

# 参考

Go Slices: usage and internals：https://blog.golang.org/go-slices-usage-and-internals
