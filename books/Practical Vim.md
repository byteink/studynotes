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



