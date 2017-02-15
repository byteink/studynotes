[toc]
# 第一部分 语言
## 第一章 类型
### 1.1 变量
Go 是静态类型语言，不能在运行期改变变量类型。
```go
var x int   // 使用关键字 var 定义变量，自动初始化为零值
var f float32 = 1.6   // 提供初始化值
var s = "abc"   // 省略变量类型，由编译器自动推断
x := 123        // 函数内部定义变量
```
也可以一次定义多个变量：
```go
var x, y, z int
var s, n = "abc", 123
var (
    a int
    b float32
)
n, s := 0x1234, "Hello world"   // 函数内
```
多变量赋值，先计算所有相关值，然后从左向右依次赋值。
特殊只写变量 "_"，用于忽略值，占位：
```go
_, s : =test()
```
编译器会将未使用的局部变量当作错误（全局变量不会），可以使用 `_ = i` 规避。
`:=`与代码层次：
```go
s := "abc"
s, y := "hello", 20   // 与前面的 s 在同一代码层次，等于对 s 重新赋值
{
    s, z := 1000, 30  // 重新定义新同名变量 s，不在同一代码层次
}
```
### 1.2 常量
常量值必须是编译期间可确定的数字、字符串、布尔值。
```go 
const x, y int = 1, 2   // 多变量初始化
const s = "Hello， world!"   // 类型推断
const (                            // 常量组
    a, b = 10, 100
    c bool = false
)
```
不支持 1UL、2LL 这样的类型后缀。未使用的局部常量不会引起编译错误。
在常量组中，如不提供类型和初始化值，那么视作与上一常量相同
```go
const (
    s = "abc"
    x    // x = "abc"
)
```
常量还可以是 `len`、`cap`、`unsafe.Sizeof`等编译器可以确定结果的函数返回值。
#### 枚举
关键字 iota 定义常量组中从 0 开始按行计数的自增枚举值
```go
const (
    Zero = iota
    One
    Two
)

const (
    _ = iota                            // iotoa = 0
    KB  int64 = 1 << (10 * iota)        // iota = 1
    MB                                  // 与KB表达式相同，但 iota = 2
    GB
    TB
)
```
在同一常量组中，可以提供多个`iota`，它们各自增长
```
const (
    A, B = iota, iota << 10    // 0, 0 << 10
    C, D                        // 1, 1 << 10
)
```
如果 `iota`自增被打断，须显示恢复
```
const (
    A = iota    // 0
    B           // 1
    C = "c"     // c
    D           // c
    E = iota    // 4, 计算包含了C、D两行
    F           // 5
)
```
可通过自定义类型来实现枚举类型限制
```go
type Color int

const (
    Black Color = iota
    Red
    Blue
)

func test(c Color) {}

c := Black
test(c)

x := 1
test(x) // 错误

test(1) // 常量会被编译器自动转换
```
### 1.3 基本类型
| 类型         | 长度    | 默认值   |说明            |
|:------------:|:---------:|:----------:|:--------------:|
| bool          | 1          |  false     |                    |
| byte          | 1          | 0            | uin8 |
| rune          | 4          | 0            | Unicode Code Point, int32 |
| int, uint      | 4 或 8  | 0            | 32 或 64位 |
| int8, uint8  | 1          | 0            | -128-127, 0 ~ 255  |
| int16, uint16 | 2       | 0            | -32768~32767, 0~65535 |
|int32,uint32 | 4         | 0            | -21亿~21亿， 0~42亿 |
|int64, uint64| 8         | 0            | |
| float32        | 4         | 0.0         ||
| float64        | 8         | 0.0         ||
| complex64 | 8         | ||
| complex128| 16      | ||   
| uintptr         | 4 或 8 | | 足以存储指针的 uint32 或 uint64 整数 |
| array           |           | | 值类型|
| struct          |            || 值类型|
|string            |           |"" |  UTF-8字符串 |
| slice             |           | nil | 引用类型    |
| map             |            | nil | 引用类型    |
| channel       |            | nil  | 引用类型   |
| interface     |             | nil   | 接口 |
| function      |             | nil   | 函数|                    

支持八进制、十六进制，以及科学计数法。标准库 math 定义了各数字类型的取值范围。
```go
a, b, c, d  := 071, 0x1F, 1e9, math.MinInt16
```
空指针为`nil`。
### 1.4 引用类型
引用类型包括`slice`、`map`和`channel`。
内置函数`new`计算类型大小，为其分配零值内存，返回指针。
而`make`会被编译器翻译成具体的创建函数，由其分配内存和初始化成员结构，返回对象而非指针。
### 1.5 类型转换
不支持隐式类型转换，即便是从窄向宽转换也不行。
```go
var b byte = 100
// var n int = b // Error
var n int = int(b)
```
同样不能将其他类型当`bool`值使用。
### 1.6 字符串
字符串是不可变值类型，内部用指针指向 UTF-8 字节数组。
- 默认值是空字符串 ""。
- 用索引号访问某字节， 如 `s[i]`
- 不能用序号获取字节元素指针，`&s[i]`非法
- 不可变类型，无法修改字节数组
- 字节数组尾部不包含 NULL

使用"`" 定义不做转义处理，支持跨行：
```go
s := `a
b\r\n\x00
c`
```
连接跨行字符串时，“+“ 必须在上一行末尾，否则导致编译错误。
```go
s := "Hello," +
        "world!"
```
支持用两个索引号返回子串。字串仍然指向原字节数组，仅修改了指针和长度属性。
```go
s := "Hello, world!"

s1 := s[:5]    // Hello
s2 := s[7:]    // world!
s3 := s[1:5]  // ello
```
单引号字符常量表示 Unicode Code Point， 支持 \uFFFF、\U7FFFFFFF、\xFF格式
对应`rune`类型，UCS-4
```go
var c1, c2 rune = '\u6211', '们'
```
要修改字符串，可先将其转换成`[]rune`或`[]byte`，完成后再转换成`string`。
无论哪种转换，都会重新分配内存，并复制字节数组。
```go
s := "abcd"
bs := []byte(s)
bs[1] = 'B'
println(string(bs))

u := "电脑"
us := []rune(s)
us[1] = '话'
println(string(us))
```
用 for 循环遍历字符串时，也有`byte`和`rune`两种方式：
```go
s:= "abc汉字"

for i := 0; i < len(s); i++ {        // byte
    fmt.Printf("%c,", s[i])
}

for _, r := range s {            // rune
    fmt.Printf("%c,", r)
}
```
###1.7 指针
支持指针类型`*T`，指针的指针`**T`，以及包含包名前缀的`*<package>.T`
- 默认值`nil`
- 操作符 "&" 取变量地址， "*"透过指针访问目标对象
- 不支持指针运算，不支持 "->"运算符，直接用 "." 访问目标成员
可以在`unsafe.Pointer`和任意类型指针间进行转换。
```go
x := 0x12345678

p := unsafe.Pointer(&x)       // *int -> Pointer
n := (*[4]byte)(p)            // Pointer -> *[4]byte

for i := 0; i < len(n); i++ {
    fmt.Printf("%X ", n[i])
} 
```
返回局部变量指针是安全的，编译器会根据需要将其分配在 GC Heap 上。
将 Pointer 转换成 uintptr，可变相实现指针运算。
```go
d := struct {
    s string
    x int 
} { "abc", 100}

p := uintptr(unsafe.Pointer(&d))
p += unsafe.Offsetof(d.x)
p2 := unsafe.Pointer(p)
px := (*int)(p2)
*px = 200

// GC把 uintptr当作普通整数对象，它无法阻止关联对象被回收。
```
###1.8 自定义类型
可将类型分为命名和未命名两大类。命名类型包括 bool、int、string等。
而 array、slice、map 等和具体元素类型、长度等有关，属于未命名类型。

具有相同声明的未命名类型被视为同一类型：
- 具有相同基类型的指针
- 具有相同元素类型和长度的 array
- 具有相同元素类型的 slice
- 具有相同键值类型的 map
- 具有相同元素类型和传送方向的 channel
- 具有相同字段序列（字段名、类型、标签、顺序）的匿名 struct
- 签名相同（参数和返回值，不包括参数名称）的 function
- 方法集相同（方法名、方法签名相同，和次序无关）的 interface

可用`type`在全局或函数内定义新类型：
```go
type bigint int64
var x bigint = 100
```
新类型不是原类型的别名，除拥有相同数据存储结构外，它们之间没有任何关系，
不会持有原类型任何信息，除非目标类型是未命名类型，否则必须显式转换。
```go
x := 1234
var b bigint = bigint(x)    // 必须显式转换，除非是常量
var b2 int64 = int64(b)    

var s myslice = []int{1, 2, 3}    // 未命名类型，隐式转换
var s2 []int = s
```
## 第2章 表达式
### 2.1 保留字
### 2.2 运算符
一元操作符号优先级最高，运算符结合律全部从左到右
binary 操作优先级：
5  `*` `/` `%` `<<` `>>` `&` `&^`
4 `+` `-` `|` `^`
3 `==` `!=` `<` `<=` `>` `>=`
2 `&&`
1 `||`

不支持运算符重载，`++`、`--`是语句而非表达式。
没有`～`，取反操作也用`^`（逻辑异或）
### 2.3 初始化
初始化复合对象，必须使用类型标签，且左大括号必须在类型尾部
```go
// var a struct { x int } = {100}  // syntax error
// var b []int = {1, 2, 3}     // syntax sugar
var a = struct {x int } {100}
var b = []int {1, 2, 3}
```
初始化值为 `,` 分隔。可以分多行，但最后一行必须以 `,` 或 `}`结尾。
```go
a := []int {
    1,
    2     // 错误
}

a := []int {
    1, 
    2,
}            // ok
    
a := [] int {
    1, 
    2 }      // ok     
```
###2.4 控制流
####2.4.1 IF
特别的写法：
- 可省略条件表达式括号
- 支持初始化语句，可定义代码块局部变量
- 代码块左大括号必须在条件表达式尾部
```go
if n := "abc"; x > 0 {   // 初始化语句未必就是定义变量，比如 println("init") 也是可以的
    println(n[2])
} else if x < 0 {
    println(n[1])
} else {
    println(n[0])
}
```
不支持三元操作符 `a > b ? a : b`
#### 2.4.2 For
支持三种循环方式，包括类 while 语法。
```go
s := "abc"

for i, n := 0, len(s); i < n; i++ {
    println(s[i])
}

n := len(s)
for  n > 0 {            // 替代 while
    println(s[n])
    n--
}

for {                    // while (true)
    println(s)
}
```
#### 2.4.3 Range
类似迭代器操作，返回（索引，值）或（键，值）
|     type          | 1st value         |       2nd value  |         note       |
|:----------------|:-------------------|:-------------------|                       |
|string            | index               | s[index]           |    unicode, rune |
|array/slice    | index               | s[index]           ||
|map              | key                 | m[key] ||
| channel        | element          |||                    
可以用`_`忽略不想要的返回值
```go
s := "abc"

for i := range s {        // 忽略 string/array/slice/map 的 2nd value
    println(s[i])
}

for _, c := range s {    // 忽略 index
    println(c)    
}

for range s {        // 忽略全部返回值。仅仅迭代
}
```
注意， `range`会复制对象
```go
a := [3]int{0, 1, 2}

for i, v := range a {                    // index、value 都是从复制品中取出
    if i == 0 {
        a[1], a[2] = 999, 999
        fmt.Println(a)                // 输出 [0, 999, 999]
    }
    
    a[i] = v + 100            // 输出 [100, 101, 102]
}
```
建议改成引用类型，其底层数据不会被复制
```go
s := []int{1, 2, 3, 4, 5}

for i, v := range s {    // 赋值 struct slice { pointer，len，cap }
    if i == 0 {
        s = s[:3]            // 对 slice 的修改不会影响 range
        s[2] = 100        // 对底层数据的修改
    }
    println(i, v)
}
```
另外两种引用类型`map`、`channel`是指针包装，而不像 `slice` 是结构体封装。
#### 2.4.4 Switch
分支表达式可以是任意类型，不限于常量，可省略 `break`，默认自动终止
如需要继续下一分支，可使用`fallthrough`，但不再判断条件。
```go
x := []int{1, 2, 3}
i := 2
switch i {
    case x[1]:
        println("a")
    case 1, 3:
        println("b")
    default:
        println("c")
}
```
省略条件表达式，可当 if-else if-else 使用
```go
switch {
    case x[1] > 0:
        println("a")
    case x[1] < 0:
        println("b")
    default:
        println("c")
}

switch i := x[2]; {    // 带初始化语句
    case i > 0:
        println("a")
    case i < 0:
        println("b")
    default:
        println("c")
}
```
#### 2.4.5 Goto, Break,Continue
支持在函数内`goto`跳转。标签名区分大小写。
配合标签，`break`和`continue`可在多级嵌套循环中跳出。
```go
func main() {
L1:
    for x := 0; x < 3; x++ {
    L2: 
        for y := 0; y < 5; y++ {
            if y > 2 { continue L2  }   
            if x > 1 {  break L1}   
            print(x, ":", y, " ")
        }   
        println()
    }   
}
```
`break` 可用于`for`、`switch`、`select`，而`continue`仅能用于`for`循环。
```go
x := 10

switch {
case x >= 0:
    if x == 0 { break }
    println(x)
}
```
## 第三章 函数
### 3.1 函数定义
不支持嵌套、重载和默认参数。
- 无需声明原型
- 支持不定长变参
- 支持多返回值
- 支持命名返回参数
- 支持匿名函数和闭包

使用关键字`func`定义函数，左大括号依旧不能另起一行。
```go
func test(x, y int, s string)(int, string) {    // 类型相同的相邻参数可以合并
    n := x + y
    return n, fmt.Sprintf(s, n)
}
```
函数是第一类对象，可作为参数传递。建议将复杂签名定义为函数类型。
```go
type FormatFunc func(s string, x, y int) string    // 定义函数类型

func test(fn func() int) int {
    return fn()
}

s1 := test(func() int { return 100 })         // 直接将匿名函数当参数
```
### 3.2 变参
变参本质上就是一个`slice`，变参只能有一个，且必须是最后一个。
```go
func test(s string, n ... int) string {
    var x int
    for _, i  := range n {
        x += i
    }
    return fmt.Sprintf(s, x)
}
```
使用`slice`对象做变参时，必须展开：
```go
s := []int{1, 2, 3}
printfln(test("sum: %d", s...))
```
###3.3 返回值
不能用容器对象接收多返回值。只能用多个变量，或“_"忽略。
```go
func test() (int, int) {
    return 1, 2
}

x, _ := test()
```
多返回值可直接作为其他函数调用实参。

命名返回参数可看做与行参类似的局部变量，最后由`return`隐式返回。
```go
func add(x, y int) (z int) {
    z = x + y
    return
}
```
命名返回参数可被同名局部变量遮蔽，此时需要显式返回。
```go
func add(x, y int) (z int) {
    {                                    // 不能在一次级别，引发 ” z redeclared in this block“ 错误
        var z = x + y
        return z                    // 必须显式返回
    }
}
```
命名返回参数允许`defer`延迟调用闭包读取和修改。
```go
func add(x, y int) (z int) {
    defer func() {
        z += 100    
    }()

    z = x + y
    return
}
```
显式 return 返回前，会先修改命名返回参数。
```go
func add(x, y int) (z int) {
    defer func() {
        println(z)            // 输出： 203
    }() 

    z = x + y 
    return z + 200         // 执行顺序： (z = z + 200)->(call defer)->(ret)                                                                
}

func main() {
    println(add(1, 2)) 
}
```
### 3.4 匿名函数
匿名函数可赋值给变量，作为结构字段，或者在`channel`里传送
```go
fn := func() { println("Hello world!") }
// function collection
fns := [](func(x int) int){
    func(x int) int { return x + 1 },
    func(x int) int { return x + 2 },
}   
println(fns[0](100))

// function as field
d := struct {
    fn func() string
}{  
   func() string { return "Hello world" },
}   
println(d.fn())

// channel of function
fc := make(chan func() string, 2)
fc <- func() string { return "Hello, world!" }
println((<-fc)())
```
闭包复制的是原对象指针，这就很容易解释延迟引用现象。
```go
func test() func() {
    x := 100
    fmt.Printf("x (%p) = %d\n", &x, x)

    return func() {
        fmt.Printf("x (%p) = %d\n", &x, x)
    }
}

func main() {
    f := test()
    f()                                                                                
}
```
在汇编层面，test实际返回的是 FuncVal 对象，其中包含了匿名函数地址、闭包对象指针。
当调用匿名函数时，只需以某个寄存器传递该对象即可。
FuncVal { func_address, closure_var_pointer ... }
### 3.5 延迟调用
关键字`defer`用于注册延迟调用。这些调用知道 ret 前才被执行，通常用于释放资源或错误处理。
多个`defer`注册，按照 FILO 次序执行。哪怕函数或某个延迟调用发生错误，这些调用依旧会被执行。

延迟调用参数会在注册时求值或复制，可用指针或闭包”延迟“读取
```go
func test() {
    x, y := 10, 20

    defer func(i int) {
        println("defer:", i, y)
    }(x)                                                                               

    x += 10
    y += 100
    println(x, y)
}
   
func main() {
    test()
}
// 输出： 20 120 
//      defer: 10 120
```
滥用`defer`可能会导致i性能问题，尤其在一个”大循环“里。
### 3.6 错误处理
没有结构化异常，使用`panic`抛出错误，`recover`捕获错误。
```go
func test() {
    defer func() {
        if err := recover(); err != nil {
            println(err.(string))        // 将interface{} 转型为具体类型
        }   
    }()                                                                                                    

    panic("panic error")
}
```
由于`panic`、`recover`参数类型为 interface{}，因此可抛出任何类型对象。
```go
func panic(v interface{})
func recover interface{}
```
`defer`调用中引发的错误，可被后续延迟调用捕获，但仅最后一个错误可被捕获。
捕获函数`recover`只有在延迟调用内直接调用才会终止错误，否则总是返回`nil`。
任何未捕获的错误都会沿调用堆栈向外传递。
```go
func test() {
    defer recover()                             // 无效                                                                   
    defer fmt.Println(recover())                // 无效
    defer func() {
        func () {
            println("defer inner")
            recover()                           // 无效
        }()
    }()

    panic("test panic")
}

func main() {
    test();
}
```
使用延迟匿名函数或者下面这样都是有效的：
```go
func except() {
    recover()
}

func test() {
    defer except()
    panic("test panic")
}
```
如果需要保护代码片段，可将代码块重构成匿名函数，如此可确保后续代码被执行。
```go
func test3(x, y int) {
    var z int
   
    func() {
        defer func() {
            if recover() != nil { z = 0 }
        }()
   
        z = x/y
        return
    }()
    println("x/y=", z)
}
```

除用`panic`引发中断性错误外，还可以返回`error`类型错误对象来表示函数调用状态。
```go
type error interface {
    Error() string
}
```
标准库`errors.New`和`fmt.Errorf`函数用于创建实现`error`接口的错误对象。
通过判断错误对象实例来确定具体错误类型。
```go
var ErrDivByZero = errors.New("division by zero")

func div(x, y int) (int, error) {
   if y == 0 { return 0, ErrDivByZero } 
   return x/y, nil
}

func main() {
    switch z, err := div(10,0); err {
    case nil:
        println(z)
    case ErrDivByZero:
        panic(err)
    }
} 
```
如何区别使用`panic`和`error`两种方式？惯例是：导致关键流程出现不可修复性错误的使用`panic`，其他使用`error`。
## 第4章 数据
### 4.1 Array
和以往认知的数组会由很大不同：
- 数组是值类型，赋值和传参会赋值整个数组，而不是指针
- 数组长度必须是常量，且是类型的组成部分。[2]int 和 [3]int 是不同类型。
- 支持 "=="、"!=“ 操作符，因为内存总是被初始化过的
- 指针数组 `[n]*T`， 数组指针`*[n]T`

可用复合语句初始化。
```go
a := [3]int{1, 2}        // 未初始化元素值为 0
b := [...]int{1, 2, 3, 4}    // 通过初始化值确定数组长度
c := [5]int{2:100, 4: 200}    // 通过索引号初始化元素

d := [...]struct {
    name string
    age uint8
} {
    {"user1", 10 },            // 可省略元素类型
    {"user2", 20},
}
```
支持多维数组：
```go
a := [2][3]int{{1, 2, 3}, {4, 5, 6}}
b := [...][2]int{{1,1}, {2, 2}, {3, 3}}    // 第2纬度不能用 ...
```
值拷贝行为会造成性能问题，通常会建议使用`slice`，或数组指针。
内置函数`len`和`cap`都返回数组长度（元素数量）。
### 4.2 Slice
`slice`并不少数组或数组指针。它通过内部指针和相关属性引用数组片段，以实现变长方案。
```c
struct Slice 
{
    byte * array;        // actual data
    uintgo len;          // number of elements
    uintgo cap;        // allocated number of elements
}
```
- 引用类型。但自身是结构体，值拷贝传递。
- 属性 `len`表示可用元素数量，读写操作不能超过该限制。
- 属性`cap`表示最大扩张容量，不能超出数组限制
- 如果 slice == nil，那么 len、cap 结果都等于 0。
```go
data := [...]int{0, 1, 2, 3, 4, 5, 6}
slice := data[1:4:5]        // [low : high: max]     len = high - low   cap = max - low
```
创建表达式使用的是元素索引号，而非数量。
读写操作实际目标是底层数组，只需要注意索引号的差别。

可直接创建 slice 对象，自动分配底层数组：
```go
s1 := []int{0, 1, 2, 3, 8:100}    // 通过初始化表达式构造，可使用索引号
s2 := make([]int, 6, 8)            // 使用 make 创建，指定 len 和 cap 值
s3 := make([]int, 6)                // 省略 cap，相当与 cap = len
```
使用`make`动态创建 slice，避免了数组必须用常量做长度的麻烦。还可以用指针访问底层数组，退化成普通数组操作。
```go
s := []int{0, 1, 2, 3}
p := &s[2]        // *int, 获取底层数组元素指针
*p += 100
```
至于`[][]T`，是指元素类型为`[]T`
#### 4.2.1 reslice
所谓 reslice 是基于已有 slice 创建新 slice 对象，以便在 cap 允许范围内调整属性。
```go
s := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}

s1 := s[2:5]         // [2, 3, 4]
s2 := s1[2:6:7]     // [4, 5, 6, 7]
s3 := s2[3:6]       // Error
```
新对象依旧指向原底层数组

#### 4.2.2 append
向 slice 尾部添加数据，返回新的 slice 对象。
简单点说，就是在 array[slice.high] 写数据。
一旦超出 slice.cap 限制，就会**重新分配**底层数组，即便原数组并未填满。

通常以 2 倍容量重新分配底层数组。在大批量添加数据时，建议一次性分配足够大的空间，以减少内存分配和数据复制操作。
或初始化足够长的 len 属性，改用索引号进行操作。及时释放不再使用的 slice 对象，避免持有过期数组，造成 GC 无法回收。
#### 4.2.3 copy
函数`copy`在两个 slice 直接复制数据，复制长度以 len 小的为准。两个 slice 可指向同一底层数组，允许元素区间重叠。
应及时将所需数据`copy`到较小的 slice，以便释放超大号底层数组内存。
### 4.3 Map
引用类型，哈希表。
键必须是支持相等运算符（==、!=）类型，比如 number、string、pointer、array、struct以及对应的 interface。
值可以是任意类型，没有限制。
```go
m := map[int] struct {
        name string
        age int
} {
       1: {"user1", 10},
       2: {"user2", 20},                                                                                  
}

println(m[1].name)
```
预先给`make`函数一个合理元素数量参数，有助于提升性能，事先申请一大块内存，避免后续操作时频繁扩展。
```go
m := make(map[string]int, 1000)
```
常见操作：
```go
m := map[string]int {
    "a" : 1,
}

if v, ok := m["a"]; ok {        // 判断 key 是否存在
    println(v)
}

println(m["4"])            // 对于不存在的 key，直接返回0, 不会出错

m["b"] = 2                 // 新增或修改

delete(m, "c")            // 删除。如果 key 不存在不会出错

println(len(m))        // 获取键值对数量， cap 无效。

for k, v := range m {        // 迭代，可仅返回key。随机顺序返回，每次都不相同
    println(k, v)
}
```
不保证迭代返回次序，通常是随机结果，具体和版本实现有关。
从 map 中取出的是一个 value 的临时复制品，对其成员的修改是没有任何意义的。
```go
type user struct{ name string }
m := map[int]user{        // 当 map 因扩张⽽重新哈希时，各键值项存储位置都会发⽣改变。 因此， map
    1: {"user1"},         // 被设计成 not addressable。 类似 m[1].name 这种期望透过原 value
}                         // 指针修改成员的⾏为⾃然会被禁⽌。

m[1].name = "Tom"         // Error: cannot assign to m[1].name
```
正确的做法是完整替换 value 或 使用指针：
```go
u := m[1]
u.name = "Tom"
m[1] = u                // 替换value

m2 := map[int]*user{
    1: &user{"user1"},
}
m2[1].name = "Jack"    // 返回的是指针复制品，透过指针修改原对象是允许的。
```
可以在迭代时安全删除键值。但如果期间有新增操作，则行为未定义。
### 4.4 Struct
```go
type Node struct {
    _ int 
    id int 
    data *byte
    next *Node
}

func main() {
    n1 := Node {
        id : 1,
        data: nil,
    }   

    n2 := Node {
        id : 2,
        data: nil,
        next: &n1,
    }                                                                                                      
}
```
顺序初始化必须包含全部字段，否则会出错。
支持匿名结构，可用作结构成员或定义变量。
```go
type File struct {
    name string
    size int
    attr struct {
        perm int
        owner int
    }
}
f := File{
    name: "test.txt",
    size: 1025,
    // attr: {0755, 1}, // Error: missing type in composite literal
}
f.attr.owner = 1
f.attr.perm = 0755

var attr = struct {
    perm int
    owner int
}{2, 0755}
f.attr = attr
```
支持“==”、“!="相等操作符，可用作 map 键类型。
可定义字段标签，用反射读取。标签是类型的组成部分。
```go
var u1 struct { name string "username"}
```
空结构”节省“内存，比如用来实现 set 数据结构，或者实现没有状态只有方法的静态类。
```go
var null struct{}

set := make(map[string]null)
```
#### 4.4.1 匿名字段
匿名字段只不过是一种语法糖，从跟本上说，就是一个与成员类型同名（不含包名）的字段。
被匿名嵌入的可以是任何类型，当然也包括指针。
```go
type User struct { name string }

type Manager struct {
    User 
    title string
}

m := Manager {
    User: User {"Tom"}
    title: "Administrator",
}
```
可以像普通字段那样访问匿名字段成员，编译器从外向内逐级查找所有层次的匿名字段，直到发现目标或出错。
```go
type Resource struct {
     id int
}

type User struct {
      Resource
      name string
}

type Manager struct {
      User
      title string                                                                                       
}

var m Manager
m.id = 1
m.name = "Jack"
m.title = "Administrator"
```
外层同名字段会遮蔽嵌入字段成员。相同层次的同名字段也会让编译器无所适从。解决办法是使用现实字段名。
不能同时嵌入某一个类型和其指针类型，因为它们名字相同。
#### 4.4.2 面向对象
面向对象三大特征里，Go 仅支持封装。尽管匿名字段的内存布局和行为类似继承。没有 class 关键字，没有继承、多态等。

## 第5章 方法
### 5.1 方法定义
方法总是绑定对象实例，并隐式将实例作为第一实参（receiver）
- 只能为当前包内命名类型定义方法
- 参数 receiver 可任意命名。如方法中未曾使用，可省略参数名。
- 参数 receiver 类型可以是 T 或 \*T。基类型 T 不能是接口或指针。。
- 不支持方法重载，receiver 只能是参数签名的组成部分
- 可用实例 value 或 pointer 调用全部方法，编译器自动转换。

没有构造和析构方法，通常用简单工厂模式返回对象实例。
```go
type Queue struct {
    elements []interface{}
}

func NewQueue() *Queue {
    return &Queue{make([]interface{}, 10)}
}

func (*Queue) Push(e interface{}) error {                              // 省略了 receiver 参数名                               
    panic("not implemented")
}

func (self *Queue) length() int {                        // receiver 参数名可以是 self、this 或其他
    return len(self.elements)
}

func main() {
    q := NewQueue()
    println(q.length())
} 
```
方法不过是一种特殊的函数，只需将其还原，就知道 receiver T 和 *T 的差别。
```go
type Data struct {
    x int
}

func (self Data) ValueTest() {                                                                             
    fmt.Printf("Pointer: %p\n", &self)
    self.x = 2
}

func (self *Data) PointerTest() {
    fmt.Printf("Pointer: %p\n", self)
    self.x = 3
}


func main() {
    d := Data{ 1 }
    p := &d
    fmt.Printf("Data: %p\n", p)

    d.ValueTest()
    d.PointerTest()

    p.ValueTest()
    p.PointerTest()
}
```
### 5.2 匿名字段
可以像字段成员那样访问匿名字段方法，编译器负责查找
```go
type User struct {
    id int 
    name string
}

type Manager struct {
    User
}

func (self *User) ToString() string {
    return fmt.Sprintf("User: %p, %v", self, self)
}

func main() {
    m := Manager{User{1, "Tom"}}

    fmt.Printf("Manager: %p\n", &m) 
    fmt.Println(m.ToString())
}  
```
通过匿名字段，可获得和继承类似的复用能力。
依据编译器查找次序，只需在外层定义同名方法，就可以实现“override”。

### 5.3 方法集
每个类型都有与之相关联的方法集，这会影响到接口实现规则。
- 类型 T 方法集包含全部 receiver T  方法。
- 类型 \*T 方法集包含全部 receiver T + \*T 方法
- 如类型 S 包含匿名字段 T，则 S 方法集包含 T 方法
- 如类型 S 包含匿名字段 \*T, 则 S 方法集包含 T + \*T 方法。
- 不管嵌入 T 或 \*T， \*S 方法集总是包含 T + \*T 方法

用实例 value 和 pointer 调用方法（含匿名字段）不受方法集约束，编译器总是查找全部方法，并自动转换 receiver 实参。
### 5.4 表达式
根据调用者不同，方法分为两种表现形式：
`instance.method(args...) `  ----->` <type>.func(instance, args...)`
前者成为 method value，后者 method expression
两者都可像普通函数那样赋值和传参，区别在于 method value 绑定实例，而 method expression 则须显示传参。
```go
type User struct {
    id int 
    name string
}

func (self *User) Test() {
    fmt.Printf("%p, %v\n", self, self)                                                                     
}  
   
func main() {
    u := User{1, "Tom"}
    u.Test()

    mValue := u.Test
    mValue()

    mExpression := (*User).Test
    mExpression(&u)
}
```
需要注意，method value 会**复制** receiver。
在汇编层面， method value 和闭包的实现方式相同，实际返回 FuncVal 类型对象。
FuncVal { method_address, receive_copy }

## 第6章 接口
### 6.1 接口定义
接口是一个或多个方法签名的集合，任何类型的方法集中只要拥有与之对应的全部方法，
就表示它“实现”了该接口，无须在该类型上显式添加接口声明。
所谓对应方法，是指有相同名称、参数列表（不包含参数名）以及返回值。
当然，该类型还可以有其他方法。
- 接口命名习惯以 er 结尾， 结构体
- 接口只有方法签名，没有实现
- 接口没有数据成员
- 可在接口中嵌入其他接口
- 类型可实现多个接口
```go
type Stringer interface {
    String() string
}

type Printer interface {
    Stringer            // 接口嵌入
    Print()
}

type User struct {
    id int 
    name string
}

func (self *User) String() string {
    return fmt.Sprintf("user %d, %s", self.id, self.name)
}

func (self *User) Print() {
    fmt.Println(self.String())
}

func main() {
    var t Printer = &User{1, "Tom"} // *User 方法集合包含String、Print
    t.Print()
} 
```
空接口 `interface {}`没有任何方法签名，也意味着任何类型都实现了空接口。
其作用类似面向对象语言中的根对象 object。
匿名接口可用作变量类型，或结构体成员。
```go
type Tester struct {
    s interface {
        String() string
    }
}

type User struct {
    id int
    name string
}

func (self *User) String() string {
    return fmt.Sprintf("user %d, %s", self.id, self.name)
}

func main() {
    t := Tester{&User{1, "Tom"}}
    fmt.Println(t.s.String())
}
```
### 6.2 执行机制
接口对象由接口表（interface table）指针和数据指针组成。
```c
struct Iface {
    Itab *tab;
    void * data;
};

struct Itab {
    InterfaceType * inter;
    Type*        type;
    void (*func[])(void);
};
```
接口表存储元数据信息，包括接口类型、动态类型，以及实现接口方法的方法指针。
无论是反射还是通过接口调用方法，都会用到这些信息。

数据指针持有的是目标对象的只读复制品，赋值完整对象或指针。
```go
type User struct {
    id int
    name string
}

func main() {
    u := User{1, "Tom"}

    var i interface{} = u

    u.id = 2
    u.name = "Jack"

    fmt.Printf("%v\n", u)
    fmt.Printf("%v\n", i.(User))
} 
```
接口转型返回临时对象，只有使用指针才能修改其状态。
```go
type User struct {
    id int
    name string
}

func main() {
    u := User{1, "Tom"}

    var vi, pi interface{} = u, &u

    // vi.(User).name = "Jack"  //Error: can not assign
    pi.(*User).name = "Jack"

    fmt.Printf("%v\n", vi.(User))  // {1 Tom}
    fmt.Printf("%v\n", pi.(*User))  // user 1, Jack                                                        
}
```
只有 tab 和 data 都为 nil 时，接口才等于 nil。
### 6.3 接口转换
利用类型推断，可判断接口对象是否某个具体的接口或类型。
```go
var o interface{} = &User{1, “Tom”}
if i，ok := o.(fmt.Stringer); ok {        // ok-idiom
    fmt.Println(i)
}
```
还可以用`switch`做批量类型判断，不支持`fallthrough`
```go
type User struct {
    id int
    name string
}

func main() {
    var o interface{} = &User{1, "Tom"}                                                                    

    switch v := o.(type) {
    case nil:
        fmt.Println("nil")
    case fmt.Stringer:
        fmt.Println(v)
    case func() string:
        fmt.Println(v())
    case *User:
        fmt.Printf("%d, %s\n", v.id, v.name)
    default:
        fmt.Println("unknown")
    }
   
} 
```
超集接口对象可转换为子集接口，反之出错。

### 6.4 接口技巧
让编译器检查，以确保某个类型实现接口。
```go
var _ fmt.Stringer = (*Data)(nil)
```
某些时候，让函数直接“实现“接口能省不少事。
```go
type Tester interface {
    Do()
}

type FuncDo func()

func (self FuncDo) Do() { self() }

func main() {
    var t Tester = FuncDo(func() { print("Hello, World!") })
    t.Do()
} 
```
## 第7章 并发
### 7.1 Goroutine
只需要在函数调用语句前添加`go`关键字，就可创建并发执行单元。调度器会自动将其安排到合适的系统线程上执行。
goroutine 是一种非常轻量级别的实现，可在单个进程里执行成千上万的并发任务。
事实上，入口函数`main`就以 goroutine 运行。

调度器不能保证多个 goroutine 执行次序，且进程退出时不会等待它们结束。
默认情况下，进程启动后仅允许一个系统线程服务于 goroutine。
可使用环境变量或标准库`runtime.GOMAXPROCS`修改，让调度器用多个线程实现多核并行。

调用`runtime.Goexit`将立即终止当前 goroutine 执行，调度器确保所有已注册 defer 调用被执行。
`runtime.Gosched`让出底层线程，将当前 goroutine 暂停，放回队列等待下次被调度执行。
### 7.2 Channel
引用类型 channel 是 CSP 模式的具体实现，用于多个 goroutine 通讯。其内部实现了同步，确保并发安全。
默认为同步模式，需要发送和接收配对。否则会被阻塞，直到另一方准备好后被唤醒。
```go
func main() {
    data := make(chan int)
    exit := make(chan bool)

    go func() {
        for d := range data {        // 从队列迭代接收数据，直到 close
            fmt.Println(d)
        }

        fmt.Println("recv over.")        // 发生退出通知
        exit <- true
    }()

    data <- 1                        // 发生数据
    data <- 2
    data <- 3
    close(data)                // 关闭队列

    fmt.Println("send over.")
    <-exit                    // 等待退出通知
} 
```
异步方式通过判断缓冲区来决定是否阻塞。如果缓冲区已满，发送被阻塞；缓冲区为空，接收被阻塞。
通常情况下，异步 channel 可减少排队阻塞，具备更高效率。但应该考虑使用指针规避大对象拷贝，将多个元素打包，减小缓冲区大小等。
缓冲区是内部属性，并非类型构成要素。
除用`range`外，还可以用 ok-idiom 模式判断 channel 是否关闭。
```go
for {
    if d, ok := <-data; ok {
        fmt.Println(d)
    } else {
        break    
    }
}
```
向 closed channel 发生数据引发 `panic`错误，接收立即返回零值。
而 ni channel，无论收发都会被阻塞。

内置函数`len`返回未被读取的缓冲元素数量，`cap`返回缓冲区大小。
#### 7.2.1 单向
可以将 channel 隐式转换为单向队列，只收或只发。
```go
c := make(chan int, 3)

var send chan<- int = c         // send-only
var recv <-chan int = c        // receive-only

send <- 1
<-recv
```
不能将单向 channel 转换为普通 channel。
#### 7.2.2 选择
如果需要同时处理多个 channel，可使用`select`语句。它随机选择一个可用 channel 做收发操作，或执行 default case。
如果没有 default case，`select`将阻塞，直到某个通信可以运行。
```go
func main() {
    a, b := make(chan int, 3), make(chan int)

    go func() {
        v, ok, s := 0, false, ""
    
        for {
            select { // 随机选择可用 channel，接收数据
            case v, ok = <-a:
                s = "a"
            case v, ok = <-b:
                s = "b"
            }

            if ok {
                fmt.Println(s, v)
            } else {
                os.Exit(0)
            }
        }
    }()

    for i := 0; i < 5; i++ {
        select { // 随机选择可用 channel，发送数据
        case a <- i:
        case b <- i:
        }
    }

    close(a)
    select {} // 没有可用 channel，阻塞 main goroutine                          
}
```
在循环中使用 select default case 需要小心，避免形成洪水。
#### 7.2.3 模式
用 `select` 实现超时（timeout）
```go
func main() {
    w := make(chan bool)
    c := make(chan int, 2)
    
    go func() {
        select {
        case v := <-c: fmt.Println(v)
        case <- time.After(time.Second * 3): fmt.Println("timeout.")
        }
        w <- true
    }()
    
    <-w
}
```
channel 是的一类对象，可传参（内部实现为指针）或者作为结构成员。
## 第8章 包
### 8.1 工作空间
编译工具对源码目录有严格要求，每个工作空间（workspace）必须由 bin、pkg、src 三个目录组成。
可在 GOPATH 环境变量列表汇总添加多个工作空间，但不能和 GOROOT 相同。
```sh
export GOPATH=$HOME/projects/golib:$HOME/projects/go
```
通常 go get 使用第一个工作空间保存下载的第三方库。
### 8.2 源文件
编码： 源码文件必须是 UTF-8 格式，否则会导致编译器出错。
结束：语句以 “；” 结束，多数时候可以省略。
注释： 支持 `//`、`/**/` 两种注释方式，不能嵌套
命名：采用 camelCasing 风格，不建议使用下划线。
### 8.3 包结构
所有代码都必须组织在 package 中。
- 源文件头部以`package <name>`声明包名称。
- 包由同一目录下的多个源码文件组成。
- 包名类似 namespace，与包所在目录名、编译文件名无关。
- 目录名最好不用 main、all、std 这三个保留名称
- 可执行文件必须包含 package main，入口函数`main`

说明：`os.Args`返回命令行参数，`os.Exit`终止进程。
            要获取正确的可执行文件路径，可用`filepath.Abs(exec.LookPath(os.Args[0]))`
包中成员以名称首字母大小写决定访问权限。
- public：首字母大写，可被包外访问
- internal：首字母小写，仅包内成员可以访问。

该规则适用于全局变量、类型、结构字段
#### 8.3.1 导入包
使用包成员前，必须使用`import`关键字导入，但不能形成导入循环。
```import "相对目录/包文件名"```
相对目录是指从`<workspace>/pkg/<os_arch>`开始的子目录，以标准库为例：
```go
import "fmt"            //  /usr/local/go/pkg/darwin_amd64/fmt.a
import "os/exec"    //  /usr/local/go/pkg/darwin_amd64/os/exec.a
```
在导入时，可指定包成员访问方式。比如对包重命名，以避免同名冲突。
```go
// test包中有个 A 成员
import M "yuhen/test"        // 包重命名: M.A
import . "yuhen/test"        // 简便模式: A
import _ "yuhen/test"        // 非导入模式：仅让该包执行初始化函数
```
对于当前目录下的子包，除使用默认完整导入路径外，还可使用 local 方式。
```go
// main.go所在目录下有个 test 目录
import "./test"     // 本地模式，仅对 go run main.go有效
```
#### 8.3.2 自定义路径
可通过 meta 设置为代码库设置自定义路径。

#### 8.3.3 初始化
初始化函数：
- 每个源文件都可以定义一个或多个初始化函数
- 编译器不保证多个初始化函数执行次序
- 初始化函数在单一线程被调用，仅执行一次。
- 初始化函数在包所有全局变量初始化后执行
- 在所有初始化函数结束后才执行 main.main
- 无法调用初始化函数

因为无法保证初始化函数执行顺序，因此全局变量应该直接用 var 初始化。
可在初始化函数中使用  goroutine，可等待其结束。
不应该滥用初始化函数，仅适合完成当前文件中的相关环境设置。
### 8.4 文档
扩展工具 godoc 能自动提取注释生成帮助文档。
- 仅和成员相邻（中间没有空行）的注释被当作帮助信息。
- 相邻行会合并成同一段落，用空行分隔段落
- 缩进表示格式化文本，比如示例代码
- 自动转换 URL 为链接
- 自动合并多个源码文件中的 package 文档
- 无法显式 package main 中的成员文档。
#### 8.4.1 Package
#### 8.4.2 Example
#### 8.4.3 Bug
## 第 9 章 进阶
### 9.1 内存布局
### 9.2 指针陷阱
### 9.3 cgo
#### 9.3.1 Flags
#### 9.3.2 DataType
#### 9.3.3 String
#### 9.3.4 Struct/Enum/Union
#### 9.3.5 Export
#### 9.3.6 Shared Library
### 9.4 Reflect











