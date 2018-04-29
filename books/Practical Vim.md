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



