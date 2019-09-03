今天开始学习下go语言，毕竟新学期开始了，也需要接触些新事物，新学期新气象，大家好好加油

# go语言结构

go语言和其他语言类似，都是由基本的几个元素组成：

- 包
- 函数
- 注释
- 语句 & 表达式
- 变量

```go
package main//定义包，必须在非注释的第一行，且每个程序都必须有个main包，表示一个可独立执行的程序

import "fmt"//编译这个程序需要使用的包，类似java中导包

//主函数，是程序开始执行的函数，main函数是每个可执行函数所必须包含的，一般是启动后的第一个执行函数，（在没有init函数的前提下）
func main()  { //{ 不能单独放在一行
	//编写函数
	fmt.Printf("hello go")//类似java中的System.Out.println();
}
```

关于包，根据本地测试得出以下几点：

- 文件名与包名没有直接关系，不一定要将文件名与包名定成同一个。
- 文件夹名与包名没有直接关系，并非需要一致。
- 同一个文件夹下的文件只能有一个包名，否则编译报错。

关于标识符

- 当标识符（包括常量、变量、类型、函数名、结构字段等等）以一个大写字母开头，如：Group1，那么使用这种形式的标识符的对象就可以被外部包的代码所使用（客户端程序需要先导入这个包），这被称为导出（像面向对象语言中的 public）；
- 标识符如果以小写字母开头，则对包外是不可见的，但是他们在整个包的内部是可见并且可用的（像面向对象语言中的 protected ）。

# 变量

go是静态类型语言，所以变量是有明确类型的，编译器会检查变量类型的正确性

## 基本类型

- bool（布尔型）
- string（字符串类型）
- int （整形）uint8,uint16,uint32,uint64
- byte（字节类型）相当于uint8
- rune(相当于uint32)
- float（浮点型）float32，float64
- complex64、complex128

## 声明变量

Go 语言的变量声明的标准格式为：

>var 变量名 变量类型

变量声明以关键字 var 开头，后置变量类型，行尾无须分号。

```go
var a int
var b float32
var c string
var d bool
var e byte
var a *int
```
懒人命名法：

```go
var (
    a int
    b string
    c []float32
)
```
简短命名法：

>变量名 := 表达式

```go
   x:=100
   a,s:=1, "abc"
```

## 变量初始化

变量声明后，每个变量会初始化其类型对应的默认值

- 整型和浮点型变量的默认值为 0。
- 字符串变量的默认值为空字符串 ""。
- 布尔型变量默认为 false
- 切片、函数、指针变量的默认为 nil。

初始化格式：

>var 变量名 类型 = 表达式

```go
//变量初始化
//标准格式
var a2 int = 1
var a3 bool = true

//编译器推导类型的格式
var a4 = 2.0
var a5 = "hello"
```

## 匿名变量

在编码过程中，可能会遇到没有名称的变量、类型或方法。虽然这不是必须的，但有时候这样做可以极大地增强代码的灵活性，这些变量被统称为匿名变量。

匿名变量的特点是一个下画线“_”，“_”本身就是一个特殊的标识符，被称为空白标识符。它可以像其他标识符那样用于变量的声明或赋值（任何类型都可以赋值给它），但任何赋给这个标识符的值都将被抛弃，因此这些值不能在后续的代码中使用，也不可以使用这个这个标识符作为变量对其它变量的进行赋值或运算。使用匿名变量时，只需要在变量声明的地方使用下画线替换即可。例如：

```go
func main()  { 

	a, _ := GetData()
	_, b := GetData()

	fmt.Println(a, b)
}

func GetData() (int, int) {
	return 100, 200
}
```

# 函数

>func 函数名 （参数列表） 返回值类型{}

```go
func imageTest()  {
	//大小
	const size  = 300

	pic :=  image.NewGray(image.Rect(0, 0, size, size))
	for x := 0 ; x < size ;x++{
		for y := 0 ; y<size;y++{
			pic.SetGray(x,y,color.Gray{255})
		}
	}

	// 从0到最大像素生成x坐标
	for x := 0; x < size; x++ {
		// 让sin的值的范围在0~2Pi之间
		s := float64(x) * 2 * math.Pi / size
		// sin的幅度为一半的像素。向下偏移一半像素并翻转
		y := size/2 - math.Sin(s)*size/2
		// 用黑色绘制sin轨迹
		pic.SetGray(x, int(y), color.Gray{0})
	}

	file , err := os.Create("test.png")

	if err != nil{
		log.Fatal(err)
	}

	png.Encode(file,pic)

	file.Close()
}
```

多返回值

```go
func testString(a,b string)(string,string){
	return a,b
}
```

# 指针

Go 语言的取地址符是 &，放到一个变量前使用就会返回相应变量的内存地址
```go
fmt.Printf("变量的地址: %x\n",&a)
```
输出：
```
变量的地址: c0000580a0
```
指针：一个指针变量指向了一个值的内存地址。

声明指针类型

```go
var a *int
```

使用
- 定义指针变量。
- 为指针变量赋值。
- 访问指针变量中指向地址的值。
```go
func main() {
   var a int= 20   /* 声明实际变量 */
   var ip *int        /* 声明指针变量 */

   ip = &a  /* 指针变量的存储地址 */

   fmt.Printf("a 变量的地址是: %x\n", &a  )

   /* 指针变量的存储地址 */
   fmt.Printf("ip 变量储存的指针地址: %x\n", ip )

   /* 使用指针访问值 */
   fmt.Printf("*ip 变量的值: %d\n", *ip )
}
```

