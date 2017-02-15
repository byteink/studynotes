# 一、breakpoint

## 设置断点

- break 16：        在源代码16行设置断点
- break func：      在函数入口处
- break +offset：
- break -offset：在当前行号的前面或后面offset行设置断点
- break filename:linenum  
- break filenam:func            指定某个文件的函数
- break *address     在程序运行的内存地址处
- break if <condition>  在条件成立时停住 例如 break if i=100
- rbreak <regex> 在符合正则的函数处设置

## 查看断点

- info break：查看断点信息
- info break n：查看第n个断点

## 对namespace内的函数设置断点
- b Foo::foo
- b (anonymous namespace)::bar      // 匿名 namespace 

## 保存断点

- save breakpoints file-name-to-save  // 保存
- source file-name-to-save                      // 载入        

## 临时断点
- tbreak                // 只触发一次就删除 简写 tb

## 断点控制

- 忽略断点
ignore bnum count     // 第 count + 1 次后才中断
- 启用/禁用断点
enable/disable bnum

# 二、watchpoint 
- watch <expr> ： 当表达式(或变量)变化时，停住程序
- rwatch <expr>：当表达式(或变量)被读时，停住程序
- awatch <expr>：被读或被写时
- watch \*(int\*)0x6009c8                     // 观察地址
- watch expr thread threadnum         // 只有编号为 threadnum 的线程改变了变量的值才中断
- info watchpoints：查看观察点

# 三、catchpoint
- catch fork          // fork调用  
- catch vfork        // vfork调用
- catch exec          // exec 调用
- 只触发一次：
          tcatch      
- 为系统调用设置 catchpoint：
    - catch syscall [name | number]         // 系统调用的编号在  /usr/share/gdb/syscalls 目录下
    - catch syscall                                         //  为所有的系统调用设置 catchpoint
    
 
# 四 打印
## 数组
### 显示大数组
在gdb中，如果要打印大数组的内容，缺省最多会显示200个元素：
更改最大限制数：
set print elements number-of-elements   // 设为 0 或 unlimited 表示不限制

### 打印数组中连续元素值

- p array[60]@10         // 打印index为60之后的 10 个元素
- p *array@10                 // 从头开始的10个元素

### 打印数组的下标
set print array-indexes on         // 默认不打印

## 打印函数局部变量
- bt full                // 显示调用栈中各个函数的局部变量
- bt full n             // 从栈顶开始显示 n 个栈帧及其局部变量
- bt full -n           // 从栈底开始显示 n 个栈帧及其局部变量
- info locals        // 显示当前函数的局部变量

## 打印进程内存信息
-  i proc mappings       // 打印进程的内存映像
-  i files                           // 更详细的内存信息，包含动态连接库

## 打印静态变量的值
p 'static-1.c'::var         // 显示地指定文件名

## 打印变量的类型和所在文件
- whatis var                  // 显示 var 的类型
- ptype var                   // 显示详细的类型信息
- i variables var            // 查看定义变量的文件 不能显示局部变量
- i variables ^var$       // 精确匹配

## 打印源代码行
- l 24               // 指定行数
- l main          // 指定函数
- l -                 // 向前打印
- l +                // 向后打印
- l 1,10             // 指定范围

## 打印结构体成员
set print pretty on    // 每成员一行，且有层次缩进

## 访问内存地址
`x/<n/f/u> <addr>`  ： 查看内存 n表示往后面取多少字节  f 表示显示格式  u表示单位
- n: 往后取多少个单位
- f：显示格式
| 格式|意义|
|:------:|:-----|
|x       | 按十六进制格式显示变量|
|d       | 按十进制格式显示变量|
|u       | 按十进制格式显示无符号整型|
|o      | 按八进制格式显示变量|
|t       |按二进制格式显示变量 |
|a      | 按十六进制格式显示变量|
|i       | 指令地址格式|
|c      | 按字符格式显示变量|
|f       | 按浮点数格式显示变量|

- u：表示一个单位是几个字节
| u  | 字节数 |
|:---:|:----|
|b|单字节|
|h|双字节|
|w|四字节|
|g|八字节|

## 按照派生类打印对象
```
Dervice d;
const Base& p = d;
```
当打印 p 时默认是按照声明的类型进行打印，设置按照派生类的类型打印：
set print object on
设置后 ptype 和 whatis 命令的结果也会改变。

## 指定程序的输入输出设备

可以为程序单独指定一个输入输出终端
1. 新开一个终端，使用 `tty` 命令获取终端名，如 /dev/pts/34
2. 指定该终端：
    - gdb -tty /dev/pts/34 ./a.out
    - 也可以在gdb命令行中输入 tty /dev/pts/34
    
## 使用 \$_ 和 \$__ 变量
`x`命令会把最后检查的地址存到 `$_` 这个变量中去，把这个地址的内容存到 `$__`这个变量中。
另外有些命令如 `info line` 和 `info address`会提供一个默认的地址给 `x` 命令检查，然后把 `$_` 改为那个值。

# 设置变量
- set var variable=expr             //  例如 set var i = 8  
- set s1 = "zzz"                           // 字符串赋值
- set {type}address=expr        // 给地址赋值 例如 set {int}0x8047a54 = 8

# 函数
## 列出函数名称
info functions [regex]
## 进入不带调试信息的函数
set step-mode on    // step命令可以进入
##  退出正在调试的函数
- finish                         // 执行完函数
- return [ret]             // 直接返回
## 直接调用函数执行
- call func()
- print func()
## 显示函数堆栈信息
- i frame                        // i 是 info 
- i registers                  // 显示寄存器信息
- disassemble func     // 反汇编函数
## 打印尾调用堆栈帧信息
set debug entry-values 1    // 设置后可以打印

# 多线程和多进程
### 调试已运行的进程
- gdb -p <pid>      // gdb启动时指定进程的ID
- attach <pid>    // 启动后在gdb命令行输入
- detach         // 结束，脱离进程


