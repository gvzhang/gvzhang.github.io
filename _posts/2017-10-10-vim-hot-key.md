---
title: Vim常用快捷键
categories:
 - 开发工具
tags:
 - vim
 - 快捷键
---

### 命令行输入
- `ctrl+a` 移动到当前行的开头
- `ctrl+e` 移动到当前行的结尾
- `ctrl+u` 擦除一行光标前面的部分
- `ctrl+h` 擦除光标前面的一个字符
- `ctrl+k` 清除光标到行尾的字符
- `ctrl+w` 清除光标之前一个单词

### vim基本用法：模式，光标移动，查找替换，复制粘贴删除
#### 帮助
- `:help`
- `:help command`

#### 模式切换
- **命令模式**     Esc, Ctrl-c, 配合光标移动可用Alt-h,Alt-j,Alt-k,Alt-l
- **编辑模式**     i 在当前位置编辑 , a在当前位置后面插入，I在行首插入，A在行尾插入，o添加新行
- **可视化模式**   v块模式，V行块模式，Ctrl-v列块模式

下面的操作方式和快捷键基本上都是在命令模式下的，编辑模式按键一般只能输入文字。

#### 输入方式
- 直接按键盘编辑     如i
- `:command`         如:set nu
- 执行shell命令     `:!command`  如:!pwd

#### 撤销,重做
- 撤销上一次的编辑操作     `u, U`
- 撤销未保存的全部编辑操作，重新载入文件 `:e!`
- 重做上一次撤销的编辑操作 `ctrl-r`
 
#### 保存，退出
- 保存文件 `:w`
- 关闭文件 `:q`
- 保存并关闭文件 `:wq`
- 不保存强制关闭文件 `:q!`
- 关闭所有文件退出 `:qa`

#### 移屏
- 下翻页 `Ctrl-f`
- 上翻页 `Ctrl-b`
- 下翻行 `Ctrl-e`
- 上翻行 `Ctrl-y`
 
#### 光标定位快捷键
- 到指定行  `:行号`，`行号G`
- 到文件头 `1G`
- 到文件尾 `G`
- 左下上右 `h,j,k,l`
- 下一个字 `w`，上一个字 `b`   
- 到行首   `^`
- 到行尾   `$`
- 行尾插入 `A`，添加空行 `o`


#### 在函数中定位光标
- `[[`  转到上一个位于第一列的“{”
- `]]`  转到下一个位于第一列的“{”
- `{`   转到上一个空行
- `}`   转到下一个空行

#### 查找当前文件
- `n,N`       查找到key后，n跳到后一个位置，N跳到前一个位置
- `* , #`     当前光标所在的词key作为关键字，精确匹配查找，相当于/\<key\>
- `g*`, `g#`    当前光标所在的词key作为关键字，忽略大小写查找，相当于/key
- `/key`      从当前光标位置开始向后查找key
- `?key`      从当前光标位置开始向前查找key
- `/\<key\>`，`?\<key\>`  精确匹配查找key
- `shift-*`  向下查找并高亮显示
- `shift-#`  向上查找并高亮显示
- `gd`    高亮显示光标所属单词，"n" 查找！
- `:nohl` 取消高亮

#### 普通查找
命令模式下，按’/’或’?’，然后输入要查找的字符，Enter。

/和?的区别是，一个向前（下）找，一个向后（上）。

#### 全词匹配
如果你输入 /int，你也可能找到 print。 
要找到以 int 结尾的单词，可以用：

```
/int\>
```

"\>" 是一个特殊的记号，表示只匹配单词末尾。类似地，"\<" 只匹配单词的开头。

一次，要匹配一个完整的单词 int，只需：

```
/\<int\>
```

#### 批量替换
- `:%s/要被取代的字串/新的字串/g`

##### replace with foo (y/n/a/q/l/^E/^Y)?
The "`y`" and "`n`" are self-explanatory, but what about the rest? To tell Vim to go ahead and replace all instances of the matched string, answer with `a`. If you realize that you don't really want to make the changes, you can tell Vim to quit the operation using `q`. To tell Vim to make the current change and then stop, use l, for last.

`^E` and `^Y` allow you to scroll the text using Ctrl-e and Ctrl-y. 

#### 复制粘贴删除
- 用`v`选中文本之后可以按`y`进行复制，如果按`d`就表示剪切，之后按`p`进行粘贴。
- 复制行 `yy`    复制n行 `nyy`
- 粘贴行 `p`
- 删除行 `dd`    删除n行 `ndd`
- 删除字 `dw`    复制字  `yw`
- 全部删除：按esc后，然后`dG`
- 全部复制：按esc后，然后`ggyG`
- `:sp [filename]`：在同一编辑窗打开第二个文件

#### 可视块选择复制：
- 进入可视化模式 `v`，`V`，`Ctrl-v`
- 可视化模式下，方向键选择块
- 按`y`复制选择的块

#### 代码折叠
- `zc` 折叠
- `zC` 对所在范围内所有嵌套的折叠点进行折叠
- `zo` 展开折叠
- `zO` 对所在范围内所有嵌套的折叠点展开
- `[z` 到当前打开的折叠的开始处。
- `]z` 到当前打开的折叠的末尾处。
- `zj` 向下移动。到达下一个折叠的开始处。关闭的折叠也被计入。
- `zk` 向上移动到前一折叠的结束处。关闭的折叠也被计入。

#### 跳转位置
- `ctrl+i` 下一个跳转位置
- `ctrl+o` 上一个跳转位置

#### vim 文件刷新
- `:e!` to force-discard your local changes and reload from the disk. 

### 配置：显示和编辑样式，配置文件

#### 显示和编辑样式
- 在状态行显示文件名`set statusline+=%f`，`set laststatus=2`
- 显示行号 `:set nu`    隐藏行号 `:set nonu`
- 自动缩进 `:set autoindent`
- c风格的缩进 `:set cindent`
- 显示断行符等特殊符号 `:set list`

### 多行注释，多文件，多窗格编辑，保存会话
#### 多行缩进：
按`v`进入`visual`状态，选择多行，用`>`或`<`缩进或缩出

#### 多行注释
多行注释按键操作：
1. **注释**：Ctrl-v 进入列编辑模式，向下或向上移动光标，把需要注释的行的开头标记起来，然后按大写的I，再插入注释符比如"#"，按Esc（两下）就会全部注释了。
2. **删除**：Ctrl-v 进入列编辑模式，向下或向上移动光标，选中注释部分，按小写d，就会删除注释符号。

多行注释使用替换命令：
1. `:%s/^/\/\//g`来在全部内容的行首添加//号注释
2. `:2,50s/^/\/\//g`在2~50行首添加//号注释
3. 反过来替换既是删除操作。

#### 删除列
1. 光标定位到要操作的地方。
2. `CTRL+v`：进入“可视块”模式，选取这一列操作多少行。
3. `d`：删除。
 
#### 插入列
插入操作的话知识稍有区别。例如我们在每一行前都插入"() "：
1. 光标定位到要操作的地方。
2. `CTRL+v`：进入“可视 块”模式，选取这一列操作多少行。
3. `SHIFT+i(I)`：输入要插入的内容。
4. `ESC`：按两次，会在每行的选定的区域出现插入的内容。

### ctags代码跳转
- `!ctags -R` 生成索引
- `:set tags=路径` 设定ctags生成的tags文件的路径
- `Ctrl-]` 跳转到唯一的或者是第一个匹配的代码
- `g+Ctrl-]` 提供多选列表
- `ctrl+t` 回到跳转之前的标签处
- `:tag {关键字}` : ex命令版的Ctrl-]
- `:tjump {关键字}`： ex命令版的g+Ctrl-]
- `:tag或:tjump /{关键字}` ：正则表达式搜索

### NERDTree目录面板
#### 切换工作台和目录
- `ctrl + w + h`    光标 focus 左侧树形目录
- `ctrl + w + l`    光标 focus 右侧文件显示窗口
- `ctrl + w + w`    光标自动在左右侧窗口切换
- `ctrl + w + r`    移动当前窗口的布局位置
- `o`       在已有窗口中打开文件、目录或书签，并跳到该窗口
- `go`      在已有窗口 中打开文件、目录或书签，但不跳到该窗口
- `t`       在新 Tab 中打开选中文件/书签，并跳到新 Tab
- `T`       在新 Tab 中打开选中文件/书签，但不跳到新 Tab
- `i`       split 一个新窗口打开选中文件，并跳到该窗口
- `gi`      split 一个新窗口打开选中文件，但不跳到该窗口
- `s`       vsplit 一个新窗口打开选中文件，并跳到该窗口
- `gs`      vsplit 一个新 窗口打开选中文件，但不跳到该窗口
- `!`       执行当前文件
- `O`       递归打开选中 结点下的所有目录
- `x`       合拢选中结点的父目录
- `X`       递归 合拢选中结点下的所有目录
- `e`       Edit the current dif
- `双击`    相当于 NERDTree-o
- `中键`    对文件相当于 NERDTree-i，对目录相当于 NERDTree-e
- `D`       删除当前书签
- `P`       跳到根结点
- `p`       跳到父结点
- `K`       跳到当前目录下同级的第一个结点
- `J`       跳到当前目录下同级的最后一个结点
- `k`       跳到当前目录下同级的前一个结点
- `j`       跳到当前目录下同级的后一个结点
- `C`       将选中目录或选中文件的父目录设为根结点
- `u`       将当前根结点的父目录设为根目录，并变成合拢原根结点
- `U`       将当前根结点的父目录设为根目录，但保持展开原根结点
- `r`       递归刷新选中目录
- `R`       递归刷新根结点
- `m`       显示文件系统菜单
- `cd`      将 CWD 设为选中目录
- `I`       切换是否显示隐藏文件
- `f`       切换是否使用文件过滤器
- `F`       切换是否显示文件
- `B`       切换是否显示书签
- `q`       关闭 NerdTree 窗口
- `?`       切换是否显示 Quick Help

#### 切换标签页
- `:tabnew` [++opt选项] ［＋cmd］ 文件      建立对指定文件新的tab
- `:tabc`   关闭当前的 tab
- `:tabo`   关闭所有其他的 tab
- `:tabs`   查看所有打开的 tab
- `:tabp`   前一个 tab
- `:tabn`   后一个 tab

标准模式下：
- `gT`           前一个 tab
- `gt`           后一个 tab
- `{i}gt`     go to tab in position i

MacVim 还可以借助快捷键来完成 tab 的关闭、切换
- `cmd+w`   关闭当前的 tab
- `cmd+{`   前一个 tab
- `cmd+}`   后一个 tab

#### NerdTree 在 .vimrc 中的常用配置
" 在 vim 启动的时候默认开启 NERDTree（autocmd 可以缩写为 au）
- `autocmd VimEnter * NERDTree`

" 按下 F2 调出/隐藏 NERDTree
- `map  :silent! NERDTreeToggle`

" 将 NERDTree 的窗口设置在 vim 窗口的右侧（默认为左侧）
- `let NERDTreeWinPos="right"`

" 当打开 NERDTree 窗口时，自动显示 Bookmarks
- `let NERDTreeShowBookmarks=1`

###  vim 撤销 回退操作
- `u`           撤销上一步的操作
- `ctrl+r` 恢复上一步被撤销的操作

### 分屏
- `ctrl+w =` ：让左右上下各个分屏宽度，高度均等
- `ctrl+w _(shift + -)` 当前屏幕高度扩展到最大
- `ctrl+w |(shift + \)` 当前屏幕宽度扩展到最大
- `ctrl+w c`：关闭当前屏幕
- `ctrl+w <`：使得当前窗口宽度减 N (默认值是 1) 
- `ctrl+w >`：使得当前窗口宽度加 N (默认值是 1)

#### 窗口大小调整
##### 纵向调整
- `ctrl+w +`：纵向扩大（行数增加）
- `ctrl+w -`：纵向缩小 （行数减少）
- `:res(ize) num`：例如：:res 5，显示行数调整为5行
- `:res(ize)+num`：把当前窗口高度增加num行
- `:res(ize)-num`：把当前窗口高度减少num行

##### 横向调整
- `:vertical res(ize) num`：指定当前窗口为num列
- `:vertical res(ize)+num`：把当前窗口增加num列
- `:vertical res(ize)-num`：把当前窗口减少num列

#### 移动分屏
- `Ctrl+W L`：向右移动
- `Ctrl+W H`：向左移动
- `Ctrl+W K`：向上移动
- `Ctrl+W J`：向下移动

### Ack代码查找
- `:Ack[!] [options] {pattern} [{directory}]`
>Search recursively in {directory} (which defaults to the current
directory) for the {pattern}.  Behaves just like the |:grep| command, but
will open the |Quickfix| window for you. If [!] is not given the first
occurrence is jumped to.

#### 打开查找结果文件
- `?`    a quick summary of these keys, repeat to close
- `o`    to open (same as Enter)
- `O`    to open and close the quickfix window
- `go`   to preview file, keeping focus on the results
- `t`    to open in new tab
- `T`    to open in new tab, keeping focus on the results
- `h`    to open in horizontal split
- `H`    to open in horizontal split, keeping focus on the results
- `v`    to open in vertical split
- `gv`   to open in vertical split, keeping focus on the results
- `q`    to close the quickfix window

### vim-php-cs-fixer代码格式化
- `<leader>`：代表键是“ , ”。**The default leader is '\', but many people prefer ',' as it's in a standard**
- `nnoremap <silent>pfd :call PhpCsFixerFixDirectory()<CR>`
- `nnoremap <silent>pff :call PhpCsFixerFixFile()<CR>`

### 切换shell
- `:sh` 进入shell
- `ctrl+d` 返回vim
- `noremap <C-d> :sh<cr>`使用`ctrl+d`来回切换shell和vim

### VIM中进行文本替换
#### 替换当前行中的内容
- `:s/from/to/`   （s即substitude）
- `:s/from/to/`    ： 将当前行中的第一个from，替换成to。如果当前行含有多个from，则只会替换其中的第一个。
- `:s/from/to/g`   ： 将当前行中的所有from都替换成to。
- `:s/from/to/gc`  ： 将当前行中的所有from都替换成to，但是每一次替换之前都会询问请求用户确认此操作。
    
    注意：这里的from和to都可以是任何字符串，其中from还可以是正则表达式。

#### 替换某一行的内容
- `:33s/from/to/g`
- `:.s/from/to/g`  ： 在当前行进行替换操作。
- `:33s/from/to/g` ： 在第33行进行替换操作。
- `:$s/from/to/g`  ： 在最后一行进行替换操作。

#### 替换某些行的内容    
- `:10,20s/from/to/g`
- `:10,20s/from/to/g`   ： 对第10行到第20行的内容进行替换。
- `:1,$s/from/to/g`    ： 对第一行到最后一行的内容进行替换（即全部文本）。
- `:1,.s/from/to/g`    ： 对第一行到当前行的内容进行替换。
- `:.,$s/from/to/g`    ： 对当前行到最后一行的内容进行替换。
- `:'a,'bs/from/to/g`   ： 对标记a和b之间的行（含a和b所在的行）进行替换。其中a和b是之前用m命令所做的标记。

#### 替换所有行的内容
- `:%s/from/to/g`
- `:%s/from/to/g`  ： 对所有行的内容进行替换。

#### 替换确认
- `:s/vivian/sky/gc`   ： 顾名思意，c是confirm的缩写

### DoxygenToolkit进行自动注释
- `:DoxAuthor`：将文件名，作者，时间等
- `:DoxLic`：license注释
- `:Dox`：函数及类注释
- `:DoxBlock`：来插入一个注释块

### 设置配色方案
- `:colorscheme solarized`：素雅 solarized
- `:colorscheme molokai`：多彩 molokai
- `:colorscheme phd`：复古 phd
