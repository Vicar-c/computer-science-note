# go

## golang的优势

1.可直接编译成机器码，不依赖其他库，因此直接运行可部署（二进制文件）

2.静态类型语言，编译的时候能检查出大部分的问题

3.语言层面的并发，这是天生的基因（底层）支持，可以充分利用多核

4.强大的标准库，包括runtime系统调度机制（执行垃圾回收及调度），以及丰富的库内容

5.简单，仅仅25个关键字，内嵌c语言语法支持，包含面向对象特征，同时跨平台



## golong适合场景

云计算基础设施：docker，kubernetes，etcd，consul，cloudflare CDN，七牛云存储等

基础后端软件：tidb（服务器），influxdb，cockroachdb

微服务：go-kit，micro，monzo bank，typhon，bilibili等

互联网基础设施：以太坊，hyperledger等



## golang的不足

1.包管理，大部分都在github上，库不稳定

2.没有泛化类型（泛型编程，在go1.18后新增）

3.所有Execption都用Error来处理（C语言也是一样，没有抛出异常）

4.对C的降级处理，并非无缝，没有C降级汇编（asm）的完美（底层的序列化问题）



## 从main函数看golang语法

```go
package main // 项目如果有main函数，那么一定有一个main包

import "fmt" // 导入的内容也是包而不是库
import "time"

// 多个包的import可以使用如下形式
/*
import(
	"fmt"
	"time"
)
*/

// 对go语言来说，“{”必须与函数名在相同行，否则编译错误
func main() {
	fmt.Printf("Hello world.\n") // 有没有分号，对go来说无所谓

	time.Sleep(1 * time.Second)
}
```

package：go语言使用包来组织源代码，实现命名空间的管理，对于main函数来说，它作为代码执行的唯一入口，只能使用main包。



## go变量声明

```go
package main

import "fmt"

func main() {
	// 变量的四种声明方式
	// 1.直接声明 默认值是0
	var a int
	// %v表示使用相应值的默认形式
	fmt.Printf("a = %v \n", a)

	// 2.声明一个变量并初始化
	var b int = 100
	fmt.Printf("b = %v \n", b)
	var bb string = "name"
	// %T表示输出其格式
	fmt.Printf("bb = %v, type of bb = %T\n", bb, bb)

	// 3.在初始化的时候省去数据类型，通过值自动匹配当前变量的数据类型
	var c = 100
	fmt.Printf("c = %v \n", c)
	fmt.Printf("type of C = %T \n", c)
	var cc = "namec"
	fmt.Printf("cc = %v, type of cc = %T\n", cc, cc)

	// 4.（最常用）省去var关键字，直接自动匹配
	// “:=” 表示既初始化，又进行赋值
	e := 100
	fmt.Printf("e = %v, type of e = %T\n", e, e)
  
  fmt.Printf("gA = %v, type of gA = %T\n", gA, gA)
	// 报错，non-declaration statement outside function body
	// fmt.Printf("gB = %v, type of gB = %T\n", gB, gB)
  
  	// 声明多个变量
	var xx, yy int = 100, 200
	var kk, ll = 100, "name"
	// 多行的多变量声明
	var (
		vv int  = 100
		jj bool = false
	)
}

// 声明全局变量(对方法1到3)
var gA int = 100
// 第四种
// gB := 100
```

前三种方式完全相同，而在全局变量的声明上，第四种方式与前三种存在区别，第四种方式只能在函数体内使用，因此不能用来声明全局变量

多变量声明与单变量存在一定差别



## const与iota关键字

```go
package main

import "fmt"

// const定义枚举类型
const (
	// BEIJING 可以在const()中添加一个关键字iota，每行的iota都会累加1，第一行的iota默认为0
	// iota的本质是自定义枚举值的增长规则，只能在const的括号中出现
	BEIJING = 10 * iota
	// SHANGHAI 此时为20
	SHANGHAI
	SHENZHEN
)

func main() {
	// 常量，只读属性
	const length int = 10
	// Println相较于Printf没有占位符
	fmt.Println("length = ", length)
}
```



## 函数多返回值的三种写法

```go
package main

import "fmt"

func fool(a string, b int) int {
	fmt.Println("a = ", a)
	fmt.Println("b = ", b)
	c := 100
	c = len(a) + b
	return c
}

// 匿名返回多个值
func fool2(a string, b int) (int, int) {
	fmt.Println("a = ", a)
	fmt.Println("b = ", b)
	return len(a), b
}

// 显示命名返回多个值,即返回值有形参名
// 可以直接对返回值赋值，返回值可以直接用来“:=”初始化
func fool3(a string, b int) (r1 int, r2 int) {
	r1 = 1000
	r2 = 2000
	return
}

// 返回值相同时，可以只写一个类型进行声明
func fool4(a string, b int) (r1, r2 int) {
	r1 = 1000
	r2 = 2000
	return
}

func main() {
	c := fool("abc", 5)
	fmt.Println("c = ", c)
	var e, f = 0, 0
	e, f = fool2("abc", 5)
	fmt.Printf("length of string = %v, int = %v \n", e, f)
	r1, r2 := fool3("abc", 5)
	fmt.Printf("length of string = %v, int = %v \n", r1, r2)
}
```

命名多返回值，还是构造了局部变量对返回值进行存储，属于该函数的形参



## import导包路径和init方法调用

go语言导包的路径先从外层进入到最底层的package，然后从底层往上依次调用init函数进行初始化

这个init函数是会在import后就执行的

```go
// test_init.go
package main

import (
  // 包名一般与目录名一致,多个文件在一个包下使用的是一个包名
  // 对外部来说，调用的方法是一样的，不需要知道具体的文件
	"Helloworld/init/include1"
	"Helloworld/init/include2"
)

func main() {
  // 不是找不到包，而是go语言规定对外暴露的函数名必须大写
	include1.L1()
	include2.L2()
}


```

import导入包必须使用（也就是调用具体的接口），不然语法会报错

这就可能出现一个情况，希望使用init而不希望使用接口，这需要匿名的导包方式

匿名导包——

1._"Helloworld/init/include1"：前面加一个下划线，表示匿名调用

2.mylib1 "Helloworld/init/include1"：使用别名，这样同时还实现了匿名的功能，允许不调用接口导包，还可以用别名调接口，例如mylib1.L1()

3.. Helloworld/init/include1"：前面加一个“.”，表示将包的所有内容导入当前包中，实现匿名，同时这样调用的时候不需要写包名，直接调用接口即可，也就是L1()

PS.不要轻易使用第三种，可能出现多个重名函数导入出现歧义的情况



## defer

```go
package main

import "fmt"

func main() {
	// 写入defer关键字
  // 这里end2先执行，defer关键字的执行顺序是先按照定义顺序压栈，然后在允许是依次出栈实现的
	defer fmt.Println("main end")
	defer fmt.Println("main end2")

	fmt.Println("hello go1")
	fmt.Println("hello go2")
}
```

defer主要用于资源的释放，它的执行在return之前，它会影响return，或者说他的核心就在与在return前对内容进行修改。

return本身不是一个原子指令，如果存在defer，那么return的值可能与它写的不一致。return这一行的执行语句和函数的return的中间执行defer，所以从体感上来说，defer执行在return执行之后



## slice

```go
package main

import "fmt"

// 数组传递,这里传递只能是一个固定值，超过或小于这个大小都不行
// 在前面加入指针也只是减少拷贝（值拷贝->引用拷贝），不能解决固定长度的问题
func printArray(myArray [4]int) {
	for index, value := range myArray {
		fmt.Println("index = ", index, "value = ", value)
	}
}

// 动态数组函数
func printArray2(myArray []int) {
	for index, value := range myArray {
		fmt.Println("index = ", index, "value = ", value)
	}
}

func main() {
	// 固定长度数组
	var myArray1 [10]int
	// 有初始化值的默认数组(需要进行至少一个值初始化，这里将前两个值分别初始化为1，2)
	myArray2 := [10]int{1, 2}
	// slice动态数组,不知道具体的大小,它本身就是指向内存的引用
	myArray := []int{1, 2, 3, 4}
	myArray3 := []int{1, 2, 3, 4}
	myArray3 = append(myArray, 10)
	// for循环
	for i := 0; i < len(myArray1); i++ {
		fmt.Println(myArray1[i])
	}

	//for i := 0; i < len(myArray2); i++ {
	//	fmt.Println(myArray2[i])
	//}

	// 使用range,会返回两个值
	for index, value := range myArray2 {
		fmt.Println("index = ", index, "value = ", value)
	}

	printArray2(myArray)
	fmt.Println("----------")
	printArray2(myArray3)
}
```

slice有四种声明方式

```go
	// slice四种声明方式
	// 1.声明var1是一个切片，初始化，默认值为1，2，3，长度为3
	var1 := []int{1, 2, 3}
	// 2.不给slice分配空间，不能直接分配，也就是直接使用索引赋值
	var var2 []int
	// 使用make进行空间开辟
	var2 = make([]int, 3)

	// 3.make结合初始化
	var3 := make([]int, 3)

	// 4.复杂一些的结构
	var var4 []int = make([]int, 3)
```















































































