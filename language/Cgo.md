# Using cgo with the go command

使用cgo需要import一个虚拟的package “C"，然后就可以引用C的类型，例如`C.size_t`。   
在`import "C"`之前的注释，称为序文(preamble)，这部分的代码在编译C部分时将被视作是一个C的头文件。    
```go
// #include <stdio.h>
// #include <errno.h>
import "C"
```

序文可以包含任何C的代码，包括函数和变量的定义、声明。可以被go代码引用。    
例外：序文的静态变量不能被引用，静态函数可以。    

CFLAGS,CPPFLAGS,CXXFLAGS,FFLAGS和LDFLAGS可以使用#cgo指令定义。     
```go
// #cgo CFLAGS: -DPNG_DEBUG=1
// #cgo amd64 386 CFLAGS: -DX86=1
// #cgo LDFLAGS: -lpng
import "C"
``` 


CPPFLAGS和LDFLAGS也可以通过pkg-config工具使用#cgo pkg-config+包名指定:   
```go
// #cgo pkg-config: png cairo
// #include <png.h>
import "C"
```


pkg-config的路径可以通过PKG_CONFIG环境变量指定。    

当解析cgo指令时，${SRCDIR}会被替换为源文件所在目录的绝对路径。    
```go
// #cgo LDFLAGS: -L${SRCDIR}/libs -lfoo
```


当Go tool看到有一个或多个go文件使用了import "C"，就会查找目录中非go文件，然后作为go package的一部分编译它们。      
.c, .s, 或者 .S将会使用c compiler编译，.cc, .cpp或者 .cxx文件将会使用C++编译器编译。.f, .F或者.f90将会使用fortran编译器编译，      
.h, .hh, .hxx文件不会独立编译，但是当头文件有修改，C和C++文件会被重新编译。     
默认的C和C++编译器可以使用CC和CXX环境变量修改。    
  
native build时cgo默认被启用，交叉编译时cgo默认是被禁用的。     
可以设置CGO_ENABLED环境变量来启用（设为1）或禁用（设为0）。    
交叉编译时，必须指定一个cgo需要的C cross-complier。    

# Go references to C  

在Go文件中，如果C的结构体成员名是Go里的关键字，可以在前面加上下划线来访问。     
如果C的struct x中有个type的成员，则可以通过x._type来访问它。    
C中的结构体中如果有成员在Go中无法表达，比如位域或者非对齐数据，会被Go忽略，并且切换为合适的padding。    

标准的C数字类型在Go中可以使用：    
C.char, C.schar(signed char), C.uchar(unsigned char),    
C.short, C.ushort, C.int, C.uint,    
C.long, C.ulong, C.longlong, C.ulonglong,    
C.float, C.double, C.complexfloat, C.complexdouble    
C中的void *会被替换为Go的unsafe.Pointer，    
C中的__int128_t和__uint128_t会被表示为[16]byte。    

访问一个struct、union 或者 enum 类型，加上前缀 struct_, union_ 或者 enum_。    

C里的任何类型T的sizof在Go中可以使用`C.sizeof_T`。       

通常情况下，Go不支持C的union类型，C的union类型会被表示为相同长度的Go的byte数组。   

Go的struct的字段不能使用C的类型。   

Go代码中不能指向出现在非空C结构体末尾的零长度字段，想要获取这些字段的地址，必须拿结构体的地址加上结构体的size。    
 
Cgo把C类型转换为等价的不可导出(unexported)Go类型。    
因此，Go的package不应该在它exported API中暴露C的类型。    
在一个Go package中使用的C类型跟另外一个go package中的相同C类型，是不同的。   

任何C的函数可以在多值赋值的上下文中调用来获取返回值（如果有）和C的errno 变量：    
如果函数返回 void，可以使用 _ 来忽略：   
```go
n, err = C.sqrt(-1)
_, err := C.voidFunc()
var n, err = C.sqrt(1)
```

调用C的函数指针限制还不支持。    
但是可以声明Go的变量持有C的函数指针，然后在C和Go之间来回传递。    
C的代码可以调用Go传递进来的函数指针。    
```go
package main

// typedef int (intFunc) ();
// 
// int bridge_int_func(intFunc f) {
//    return f();    
// }
// 
// int fortytwo() {
//    return 42;
// }

import "C"
import "fmt"

func main() {
    f := C.intFunc(C.fortytwo)
    fmt.Println(int(C.bridge_int_func(f)))
    // output : 42
}
```

在C中，一个函数的参数可以是固定长度的数组(实际上需要的是指向这个数组第一个元素的指针)。      
但是在Go中，必须显式地传递指向第一个元素的指针：C.f(&C.x[0])。     

一些特殊的使用拷贝方式来转换C和Go类型的函数：      
```go
// Go string -> C string 
// C string是使用malloc的来分配的
// 调用者负责释放（include stdlib.h, 调用C.free）
func C.CString(string) *C.char


// Go []byte -> C array
// C array使用了malloc分配，调用者负责释放
func C.CBytes([]byte) unsafe.Pointer


// C string -> Go string
func C.GoString(*C.char) string

// C 固定长度的数据 -> Go string
func C.GoStringN(*C.char, C.int) string

// C 固定长度数据 -> Go []byte
func C.GoBytes(unsafe.Pointer, C.int) []byte
```

一个特例，C.malloc不直接调用C库的malloc，而是封装了一下保证绝不返回nil。     
如果C的malloc内存不足，程序就会crash。    

# C references to Go

Go的函数可以被导出在C代码中使用：  
```go
//export MyFunction
func MyFunction(arg1, arg2 int, arg3 string) int64 {...}

//export MyFunction2
func MyFunction2(arg1, arg2 int, arg3 string) (int64, *C.char) {...}
```

这将会在C中表示如下形式：
```c
extern int64 MyFunction(int arg1, int arg2, GoString arg3);
extern struct MyFunction2_return MyFunction2(int arg1, int arg2, GoString arg3);
```

可以在生成的 _cgo_export.h 中找到这些声明。    
多返回值的函数会被转成返回一个struct的函数。    

# Passing pointers

Go有GC，GC需要知道每个指向Go内存的指针。因此Go和C之间的指针传递会有所限制。   

Go可以传递一个Go的指针给C，这个Go的指针指向的Go内存需要不包含Go指针。    
C代码必须保证：它不能在Go的内存中存储Go的指针，暂时地也不行。    
当把指针传递给一个struct的一个字段，这个Go内存属于字段，而不是整个struct。    

C代码在调用返回后不可以继续持有Go指针的拷贝。    

一个被C调用的Go函数不可以返回一个Go的指针。    
一个被C调用的Go函数可以用C指针作为参数，可以通过这些指针存储非指针或者C指针数据，但是不能存储Go指针。    
一个被C调用的Go函数可以用Go指针作为参数，但是必须保证这个Go指针指向的Go内存中不包含其他Go指针。     

Go代码不可以在C内存中存储Go的指针。    
C代码可以在C内存中存储Go的指针，但必须服从上面的规则：当C函数返回时必须停止存储Go指针。    

这些规则可以在运行时动态地检查，通过GODEBUG环境变量的cgocheck设置。    
默认设置是GODEBUG=cgocheck=1，实现合理的较轻便的检查。可以通过GODEBUG=cgocheck=0来禁用。    
完整的检查：GODEBUG=cgocheck=2, 但是会有一些开销。    

# Using cgo directly

用法：  
```shell
go tool cgo [cgo options] [-- compiler options] gofiles...
```

cgo会把指定的输入（Go源文件）成若干输出（Go和C源文件）。    

compiler选项会不动地传递给C compiler当编译package的C部分时。     
