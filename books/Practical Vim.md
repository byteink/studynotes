# Practical Vim
## Chapter 1 The Vim Way
### 1. Meet the Dot Command
dot命令可以使我们重复最近的修改操作（repeats the last change）。     
修改操作可以是单个字符级别的、行级别的甚至整个文件级别的。    
例如 x 命令删除光标下的字符，dd命令删除整行。    
Insert模式下的修改也可以适用。    

### 2. Don't Repeat Yourself
vim很多单字符命令都可以看作是两个或多个命令的组合。    
如`C`等同于`c$`, `s`等同于`cl`，`O`等同于`ko`，`S`等同于`^C`   

### 3. Take One Step Back, Then Three Forward

### 4. Act、Repeat、Reverse
`f{char}`/`t{char}`：行内查找向右搜索字符      
`F{char}`/`T{char}`：行内查找向左搜索字符      
`/pattern<CR>`: 文档下向下搜索模式       
`?pattern<CR>`: 文档下向上搜索模式
`&`: 重复上一次替换操作      
`@`: 重复任何上一次命令,后面可跟参数，比如`@:`    
`qx{change}`: 执行一系列修改，然后`@x`可以重复    

### 5. Find and Replace by Hand
`cw`: 删除到词末，然后进入insert模式。
### 6. Meet the Dot Formula
## Chapter 2 Normal Mode
### 7. Pause with Your Brush Off the Page
### 8. Chunk Your Undos
vim可以控制undo命令的粒度。   
从进入Insert模式到返回Normal模式，期间的所有修改会看作成**一次**change。                
因此可以通过返回Normal模式来控制undo的粒度。     
> 在Insert模式下使用方向键，也会重置undo chunk。    
> 因为方向键可以看作我们返回了Normal mode然后使用了h,j,k,l等命令。

### 9. Compose Repeatable Changes
`db`命令：从光标处删除到词首，`dw`删除到词尾    
`daw`：删除整个词，不管光标在词的哪个位置。    
### 10. Use Counts to Do Simple Arithmetic
大多数Normal mode下的命令可以附带额外的count操作。    
      
`<C-a>`和`<c-x>`可以在数字上执行增减操作，可以在命令前加一个数字 n 来增减 n。    
如果当前光标位置不是一个数字，会对光标后面的数字执行。     
诸如`007`这样的格式会按做八进制来增减，可以`set nrformats-=octal`来避免。    
### 11. Don't Count if You Can Repeat
诸如`2dw`与`dw.`我们倾向于后者，因为后者粒度更小。而且带count的容易出错，比如输错数字。   
### 12. Combine and Conquer
`d{motion}`：可以是`dl` `daw` `dap`等，`d`称为operator，它作用的范围取决于后面的motion。       
其他的operators还有`c`、`y`、`g~`、`gu`、`gU`、`>`、`<`、`=` 、`!`。         
它们都遵循相同的 motion 语法。例如两次相同的operator代表作用于当前行。如`dd` `>>`           
         
`cf{char}`、`ct{char}`：删除到行内`{char}`处，这也是一个motion     
        
我们还可以自定义自己的 operator 和 motion。see `:help map-operator`。        

## Chapter 3 Insert Mode
### 13. Make Correction Instantly from Insert Mode
`<C-h>`: 回删一个字符      
`<C-w>`：回删一个word      
`<c-u>`：回删至行首       
### 14. Get Back to Normal Mode
除了`<Esc>`键还可以通过`<C-[>`切换到Normal Mode。   
    
`<C-o>`：切换到Insert-Normal mode。  
可执行一次Normal mode命令，然后自动切换insert mode。   
### 15. Paste from a Register Without Leaving Insert Mode
`<C-r>0`：插入模式下在当前光标位置插入之前yank的文本。     
`<C-r>{register}`: 后面可以跟不同的register地址。      
`<C-r><C-p>{register}`: 原封不动地粘贴，不会添加额外的换行和缩进。    

### 16. Do Back-of-the-Envelop Calculations in Place
delete 和 yank 命令可以使我们设置一个 register 的内容，然后用put命令可以插入到文档。    
     
expression register：    
可以执行计算，也可以是一段vim脚本代码，然后直接插入到文档。    
在insert mode下可以键入`<C-r>=`来访问。    

### 17. Insert Unusual Characters by Character Code
在insert mode下，输入`<C-v>{code}`可以输入任意字符。`{code}`是想要插入字符的地址。    
地址可以是三个数字组成，如`065`是`A`。也可以是`u00bf`unicode字符。    
使用`ga`命令可以显示当前字符的code。       
     
如果`<C-v>`命令跟的是非数字，会直接插入这个字符。          
并且如果我们跟的是`<Tab`键，会忽略`expandtab`配置，插入tab而不是空格。      

### 18. Insert Unusual Characters by Digraph
`<C-k>{char1}{char2}`插入二合字母。    
例如 `<<`表示 « ，`12` 表示 ½。      
执行`:digraphs`可以列出可用的二合字母。       

### 19. Overwrite Existing Text with Replace Mode
Replace mode 跟 insert mode 是一样的，除了会覆盖文档中已存在的文本。         
从 Normal mode 可以输入`R`来进入 Replace Mode。     
       
`gR`进入 Virtual Replace Mode，会把 tab 看作一个字符来进行替换而不是多个空格。      
    
一次性的 Replace Mode：   
`r{char}`和`gr{char}`替换一个字符然后返回Normal mode。  



## Chapter 4  Visual Mode
### 20. Grok Visual Mode
Visual Mode 下许多命令跟 Normal mode一样：      
如果可以使用`h`,`j`,`k`,`l`；    
可以使用`f{char}`及`;`和`,`来跳转；   
可以使用搜索命令及`n`/`N`；

`<C-g>`： 在 Visual Mode 和 Select Mode 之间切换。    
Select Mode 下输入任何可打印字符将会删除选中内容，并进入 Insert Mode。      
这是类似普通文本编辑器的形式，在vim中应该会比较少使用。 
   
### 21. Define a Visual Selection
`v`: character-wise Visual mode     
`V`: line-wise Visual mode        
`<C-v>`: block-wise Visual mode      
`gv`: 重新选择上一次的选择的 visual mode 范围         
    
处于 Visual Mode时：
可以键入进入时相同的命令，可以返回 Normal mode。比如两次`v`        
也可以再进入不同 wise 的 visual mode。比如 `v` 然后 `V`     

`o`: 在选中块起始和结束位置之间跳转。     
我们可以使用`o`来更改选中块的范围（跳转到边界再移动增加范围）。    

### 22. Repeat Line-Wise Visual Commands
dot命令会在相同 visual selection 上重复修改命令。

### 23. Prefer Operators to Visual Commands Where Possible
`it`: 特殊的 motion， inside the tag。    
     
 `U`: visual mode 下转化选中的字符为大写

### 24. Edit Tabular Data with Visual-Block Mode
`r{char}`: 在 visual mode 下，会把所有选中部分替换为同一个字符。

### 25. Change Columns of Text
在 Visual-Block mode下执行 insert， 当`<Esc>`回到 Normal mode 时，insert会执行到多行。

### 26. Append After a Ragged Visual Block
Visual-Block mode下:      
`$`：跳到每行的末尾。 比如后面再`A`可以实现在所有行后面添加

## Chapter 5 Command-Line Mode

### 27. Meet Vim's Command Line

可以操作Buffer里文本的Ex Command：      
`:[range]delete [x]`：删除指定行[到 register x]      
`:[range]yank [x]`: yank指定行到[register x]           
`:[line]put [x]`: put register x 中的文本到指定行后面    
`:[range]copy{address}`: copy 指定行到 {address} 行的下面    
`:[range]move{address}`: move 指定行到 {address} 行的下面           
`:[range]join`: join指定行    
`:[range]normal {commands}`：对指定的每一行执行 normal-mode 命令    
`:[range]substitue/{pattern}/{string}/[flags]`: 替换    
`:[range]global/{pattern}/[cmd]`: 对指定行符合模式的执行 ex command

当键入`:`， vim 就切换到 Command-Line mode。   
因为历史原因，在这一模式下执行的命令称为 Ex Commands。    
`/`(search) 和 `<C-r>=`(expression register)也会进入这一模式。

### 28. Execute a Command on One or More Consecutive Lines
#### 使用行号作为地址
`:p`或者`:print` 打印当前行     
`:1`跳到第一行    
`:$`跳到文件尾     
`:3p` 打印第三行    
自然也可以后跟其他命令，比如`:3d` 删除第三行     
#### 指定一个行范围
`:2,5p` 打印2到5行，格式 :{start},{end}    
`:%p` %号表示文件内所有行，等价于`:1,$p`  
`:.,$p` .号表示当前行  

#### 使用Visual Selection 指定行范围

#### 使用模式匹配指定行范围    
`/<html>/,/<\/html>/p`: 用模式匹配指定起始行和结束行。
#### 使用Offset修改地址
格式 :{address}+n，n可以忽略默认是1. {address}可以是行号、mark或者pattern，    
例如：     
`:/<html>/+1, /<\/html>/-1p`        
`:.,.+3p` 打印当前行及其后的三行

#### 用于指定地址和范围的符号总结
`1` 第一行    
`$` 最后一行    
`0` 虚拟的，表示第一行之上    
`.` 光标所在当前行     
`,m` 包含 mark m 的行    
`'<` visual selection 起始行    
`'>` visual selection 结束号    
`$` 所有行    

我们制定的 [range] 都是连续的行，可以使用 :global 命令制定不连续的行。
   
### 29. Duplicate or Move Lines Using ':t' and ‘:m' Commands
`:t`是`:copy`的缩写形式    
`:m`是`:move`的缩写形式      

   
`:6copy.`：把第6行拷贝到当前行。  
`:t$`: 拷贝当前行到文件尾，不指定前面的 [range] 表示是拷贝当前行。     
`'<,'>t0`：visual mode下选中的行拷贝到文件首   
         
copy不会像 yank 命令占用 register。    
  
`'<,'>m$`移动到文件尾。     
     
`@:` 重复上一次 ex command。

### 30. Run Normal Mode Commands Across a Range
`:'<,'>normal .` 对选中行重放上次修改操作    
`:%normal A;` 在所有行尾添加 ; 号    
`:%normal i//` 在所有行首添加 // (注释）

这对同时修改多行非常有用！

### 31. Repeat the Last Ex Command
`.`命令不能重放Command line命令的修改，需要使用`@:`    
最常用的比如重放`:bn[ext]`或者`:bp[revious]`。      

这跟重放宏是类似的，`:`是个特殊的register，保存最近执行的command line命令。    
运行一次`@:`后，后续可以使用`@@`来重放。   

### 32. Tab-Complete Your Ex Commands
可以使用`<Tab>`键来补全 Ex Command。    
`<C-d>`：显示可能的补全   
`<Tab>`：选中下一个候选补全    
`<S-Tab>`：选中上一个候选补全     
    
我们也可以对我们自定义的Ex Command进行补全行为。    

#### 从多个匹配中挑选
wildmode 配置可以定制补全模式。    
bash shell 一般配置为：   
       
```sh
set wildmode=longest,list
```  
        
如果你习惯zsh提供的autocomplete menu，可以用下面的配置：     
     
```sh
set wildmenu              
set wildmode=full
```       
   
当 wildmenu 启用后，可以使用`<Tab>`,`<C-n>`,`<Right>`键选择下一个；    
使用`<S-Tab>`,`<C-p>`,`<Left>`选择上一个。

### 33. Insert the Current Word at the Command Prompt
`<C-r><C-w>`：拷贝光标下的word插入到 command-line。    
### 34. ReCall Commands from History
按下`:`键切换到Command-Line mode后，使用`<Up>`和`<Down>`可以翻阅历史命令。    
如果我们输入`:help`, 再使用`<Up>`或`<Down>`翻阅的只是以 help 开头的历史命令。      

默认只保存20条历史命令，可以`set history=200`来配置。    
另外这个历史保存不只是当前会话的，而是会持久化，退出再重启后也还会在。   

vim也会保存 search 历史，输入`/`来翻阅它。    

#### Command-Line Window
command-line window很像一个普通的vim buffer。每行一个历史条目。    
输入`q:`命令可以打开command-line window。    
使用`k`和`j`可以在历史条目中上下移动。按下`<CR>`回车键会执行当前历史。   
使用`:q`可以关闭它。      
    
我们还可以编辑修改历史命令。    
可以在Normal-mode下跳转，在visual-mode下操作或者切换到insert-mode。    
设置还可以对这些历史执行 Ex commands。   

`q/` 打开search历史的command-line window         
`<C-f>` 从 Command-Line mode 转换到 command-line window    

### 35. Run Commands in the Shell
command-line mode下前面加上 ! 号来执行外部 shell 命令。     
`:!echo %`：% 号代表当前文件名。      
        
`:shell`： 打开一个交互 shell 会话。退出后返回vim。     

#### 使用vim buffer 作为外部命令的标准输入或输出
`:read !{cmd}`：将 {cmd} 的输出写入当前buffer。     
`:write !{cmd}`：将当前buffer的内容作为命令的标准输入。      
`:[range]write !{cmd}`：指定作为标准输入的文本范围。      
注意与`:wirte!`的区别：    
`:write! sh`：把当前buffer覆盖写入一个名为 sh 的文件里。

#### 通过外部命令过滤 buffer 里的内容
`:!{cmd}`可以指定一个范围 [range]，`:[range]!{cmd}`        
这时 [range] 内的文本将会作为 {cmd} 的标准输入，   
并且命令的输入会替换 [range] 内的文本。  
        
这可以实现将vim里的文本通过外部程序处理修改后再写回。     
例如 `:2,$!sort -t',' -k2`：按第二列排序文本。     

`!{motion}`操作符可以切换到Command-Line mode并且提前计算好 motion 对应的 [range]。      
例如`!ap`，`!2j`。    


### 36. Run Multiple Ex Commands as a Batch
可以把多条Ex commands写入到一个文件内，比如 batch.vim     
然后使用`:source batch.vim`来批量执行。   
文件内的每一行都将当作一条 Ex command 来执行。  

#### 更改多个文件
`argdo source batch.vim`：对argument list里每个文件执行批量操作。

## Chapter 6 Manage Multiple Files
### 37. Track Open Files with the Buffer List

### 38. Group Buffers into a Collection with the Argument List
### 39. Manage Hidde Files
### 40. Divide Your Workspace into Split Windows
### 41. Organize Your Window Layouts with Tab Pages

## Chapter 7 Open Files and Save Them to Disk


