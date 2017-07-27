原文：[强大的vim配置文件，让编程更随意](http://www.cnblogs.com/ma6174/archive/2011/12/10/2283393.html)

该vim配置主要有以下优点：

1. 按`F5`可以直接编译并执行`C`、`C++`、`java`代码以及执行`shell`脚本，按`F8`可进行`C`、`C++`代码的调试
2. 自动插入文件头 ，新建`C`、`C++`源文件时自动插入表头：包括文件名、作者、联系方式、建立时间等，读者可根据需求自行更改
3. 映射`Ctrl+A`为全选并复制快捷键，方便复制代码
4. 按`F2`可以直接消除代码中的空行
5. `F3`可列出当前目录文件，打开树状文件目录
6. 支持鼠标选择、方向键移动
7. 代码高亮，自动缩进，显示行号，显示状态行
8. 按`Ctrl+P`可自动补全
9. `[]`、`{}`、`()`、`""`、`''`等都自动补全
10. 其他功能读者可以研究以下文件

若读者使用其他编程语言，可以根据自己的需要进行修改，配置文件里面已经加上注释。

### 效果图

![](超强vim配置文件[转]/1.png)

### 完整安装

具体详见：https://github.com/ma6174/vim

#### 简易安装

打开终端，执行下面的命令就自动安装好了：
```bash
wget -qO- https://raw.github.com/ma6174/vim/master/setup.sh | sh -x
```

#### 手动安装（以ubuntu为例）

1. 安装vim
```bash
sudo apt-get install vim
```
2. 安装ctags
```bash
sudo apt-get install ctags
```
3. 安装一些必备程序
```bash
sudo apt-get install xclip vim-gnome astyle python-setuptools
```
4. python代码格式化工具
```bash
sudo easy_install -ZU autopep8
sudo ln -s /usr/bin/ctags /usr/local/bin/ctags
```
5. clone配置文件
```bash
cd ~/ && git clone git://github.com/ma6174/vim.git
mv ~/vim ~/.vim
mv ~/.vim/.vimrc ~/
```
6. clone bundle 程序
```bash
git clone https://github.com/gmarik/vundle.git ~/.vim/bundle/vundle
```
7. 打开vim并执行bundle程序
```vim
:BundleInstall
```
8. 重新打开vim即可看到效果

#### 使用提示

**编写python程序**

* 自动插入头信息：
```py
#!/usr/bin/env python
# coding=utf-8
```
* 输入`.`或按`TAB`键会触发代码补全功能
* `:w`保存代码之后会自动检查代码错误与规范
* 按`F6`可以按pep8格式对代码格式优化
* 按`F5`可以一键执行代码

**多窗口操作**

* 使用`:sp+文件名`可以水平分割窗口
* 使用`:vs+文件名`可以垂直分割窗口
* 使用`Ctrl+w`可以快速在窗口间切换

**编写markdown文件**

* 编写markdown文件（*.md）的时候，在`normal`模式下按`md`即可在当前目录下生成相应的html文件
* 生成之后还是在`normal`模式按`fi`可以使用`firefox`打开相应的html文件预览
* 当然也可以使用万能的`F5`键来一键转换并打开预览
* 如果打开过程中屏幕出现一些混乱信息，可以按`Ctrl+l`来恢复

**快速注释**

* 按`\`可以根据文件类型自动注释

**主题更换**

* 400多种主题，可以在[https://github.com/ma6174/vim/tree/master/colors](https://github.com/ma6174/vim/tree/master/colors)目录中找到
* 可以在[http://vimcolors.com/](http://vimcolors.com/)预览
* 将`.vimrc`文件中`color ron`中的`ron`换成你喜欢的主题名字即可
* 重新打开vim生效

### 精简安装

下面是精简的，没有插件的vim配置文件，保存到自己的.vimrc文件或直接复制下面的代码到文本文件，然后把文件改名为`.vimrc`（不要忘记前面的`.`），然后把文件放到用户文件夹的根目录下面即可。重新打开vim即可看到效果。
```vim
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
" 显示相关
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

"set shortmess=atI   " 启动的时候不显示那个援助乌干达儿童的提示

"winpos 5 5          " 设定窗口位置

"set lines=40 columns=155    " 设定窗口大小

"set nu              " 显示行号

set go=             " 不要图形按钮

"color asmanian2     " 设置背景主题

set guifont=Courier_New:h10:cANSI   " 设置字体

"syntax on           " 语法高亮

autocmd InsertLeave * se nocul  " 用浅色高亮当前行

autocmd InsertEnter * se cul    " 用浅色高亮当前行

"set ruler           " 显示标尺

set showcmd         " 输入的命令显示出来，看的清楚些

"set cmdheight=1     " 命令行（在状态行下）的高度，设置为1

"set whichwrap+=<,>,h,l   " 允许backspace和光标键跨越行边界(不建议)

"set scrolloff=3     " 光标移动到buffer的顶部和底部时保持3行距离

set novisualbell    " 不要闪烁(不明白)

set statusline=%F%m%r%h%w\ [FORMAT=%{&ff}]\ [TYPE=%Y]\ [POS=%l,%v][%p%%]\ %{strftime(\"%d/%m/%y\ -\ %H:%M\")}   "状态行显示的内容

set laststatus=1    " 启动显示状态行(1),总是显示状态行(2)

set foldenable      " 允许折叠

set foldmethod=manual   " 手动折叠

"set background=dark "背景使用黑色

set nocompatible  "去掉讨厌的有关vi一致性模式，避免以前版本的一些bug和局限

" 显示中文帮助

if version >= 603

    set helplang=cn

    set encoding=utf-8

endif

" 设置配色方案

"colorscheme murphy

"字体

"if (has("gui_running"))

"   set guifont=Bitstream\ Vera\ Sans\ Mono\ 10

"endif



set fencs=utf-8,ucs-bom,shift-jis,gb18030,gbk,gb2312,cp936

set termencoding=utf-8

set encoding=utf-8

set fileencodings=ucs-bom,utf-8,cp936

set fileencoding=utf-8



"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
"""""新文件标题
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

"新建.c,.h,.sh,.java文件，自动插入文件头

autocmd BufNewFile *.cpp,*.[ch],*.sh,*.java exec ":call SetTitle()"

""定义函数SetTitle，自动插入文件头

func SetTitle()

    "如果文件类型为.sh文件

    if &filetype == 'sh'

        call setline(1,"\#########################################################################")

        call append(line("."), "\# File Name: ".expand("%"))

        call append(line(".")+1, "\# Author: ma6174")

        call append(line(".")+2, "\# mail: ma6174@163.com")

        call append(line(".")+3, "\# Created Time: ".strftime("%c"))

        call append(line(".")+4, "\#########################################################################")

        call append(line(".")+5, "\#!/bin/bash")

        call append(line(".")+6, "")

    else

        call setline(1, "/*************************************************************************")

        call append(line("."), "    > File Name: ".expand("%"))

        call append(line(".")+1, "    > Author: ma6174")

        call append(line(".")+2, "    > Mail: ma6174@163.com ")

        call append(line(".")+3, "    > Created Time: ".strftime("%c"))

        call append(line(".")+4, " ************************************************************************/")

        call append(line(".")+5, "")

    endif

    if &filetype == 'cpp'

        call append(line(".")+6, "#include<iostream>")

        call append(line(".")+7, "using namespace std;")

        call append(line(".")+8, "")

    endif

    if &filetype == 'c'

        call append(line(".")+6, "#include<stdio.h>")

        call append(line(".")+7, "")

    endif

    "新建文件后，自动定位到文件末尾

    autocmd BufNewFile * normal G

endfunc

""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
"键盘命令
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""



nmap <leader>w :w!<cr>

nmap <leader>f :find<cr>



" 映射全选+复制 ctrl+a

map <C-A> ggVGY

map! <C-A> <Esc>ggVGY

map <F12> gg=G

" 选中状态下 Ctrl+c 复制

vmap <C-c> "+y

"去空行

nnoremap <F2> :g/^\s*$/d<CR>

"比较文件

nnoremap <C-F2> :vert diffsplit

"新建标签

map <M-F2> :tabnew<CR>

"列出当前目录文件

map <F3> :tabnew .<CR>

"打开树状文件目录

map <C-F3> \be

"C，C++ 按F5编译运行

map <F5> :call CompileRunGcc()<CR>

func! CompileRunGcc()

    exec "w"

    if &filetype == 'c'

        exec "!g++ % -o %<"

        exec "! ./%<"

    elseif &filetype == 'cpp'

        exec "!g++ % -o %<"

        exec "! ./%<"

    elseif &filetype == 'java'

        exec "!javac %"

        exec "!java %<"

    elseif &filetype == 'sh'

        :!./%

    endif

endfunc

"C,C++的调试

map <F8> :call Rungdb()<CR>

func! Rungdb()

    exec "w"

    exec "!g++ % -g -o %<"

    exec "!gdb ./%<"

endfunc

""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
""实用设置
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

" 设置当文件被改动时自动载入

set autoread

" quickfix模式

autocmd FileType c,cpp map <buffer> <leader><space> :w<cr>:make<cr>

"代码补全

set completeopt=preview,menu

"允许插件

filetype plugin on

"共享剪贴板

set clipboard+=unnamed

"从不备份

set nobackup

"make 运行

:set makeprg=g++\ -Wall\ \ %

"自动保存

set autowrite

set ruler                   " 打开状态栏标尺

set cursorline              " 突出显示当前行

set magic                   " 设置魔术

set guioptions-=T           " 隐藏工具栏

set guioptions-=m           " 隐藏菜单栏

"set statusline=\ %<%F[%1*%M%*%n%R%H]%=\ %y\ %0(%{&fileformat}\ %{&encoding}\ %c:%l/%L%)\

" 设置在状态行显示的信息

set foldcolumn=0

set foldmethod=indent

set foldlevel=3

set foldenable              " 开始折叠

" 不要使用vi的键盘模式，而是vim自己的

set nocompatible

" 语法高亮

set syntax=on

" 去掉输入错误的提示声音

set noeb

" 在处理未保存或只读文件的时候，弹出确认

set confirm

" 自动缩进

set autoindent

set cindent

" Tab键的宽度

set tabstop=4

" 统一缩进为4

set softtabstop=4

set shiftwidth=4

" 不要用空格代替制表符

set noexpandtab

" 在行和段开始处使用制表符

set smarttab

" 显示行号

set number

" 历史记录数

set history=1000

"禁止生成临时文件

set nobackup

set noswapfile

"搜索忽略大小写

set ignorecase

"搜索逐字符高亮

set hlsearch

set incsearch

"行内替换

set gdefault

"编码设置

set enc=utf-8

set fencs=utf-8,ucs-bom,shift-jis,gb18030,gbk,gb2312,cp936

"语言设置

set langmenu=zh_CN.UTF-8

set helplang=cn

" 我的状态行显示的内容（包括文件类型和解码）

"set statusline=%F%m%r%h%w\ [FORMAT=%{&ff}]\ [TYPE=%Y]\ [POS=%l,%v][%p%%]\ %{strftime(\"%d/%m/%y\ -\ %H:%M\")}

"set statusline=[%F]%y%r%m%*%=[Line:%l/%L,Column:%c][%p%%]

" 总是显示状态行

set laststatus=2

" 命令行（在状态行下）的高度，默认为1，这里是2

set cmdheight=2

" 侦测文件类型

filetype on

" 载入文件类型插件

filetype plugin on

" 为特定文件类型载入相关缩进文件

filetype indent on

" 保存全局变量

set viminfo+=!

" 带有如下符号的单词不要被换行分割

set iskeyword+=_,$,@,%,#,-

" 字符间插入的像素行数目

set linespace=0

" 增强模式中的命令行自动完成操作

set wildmenu

" 使回格键（backspace）正常处理indent, eol, start等

set backspace=2

" 允许backspace和光标键跨越行边界

set whichwrap+=<,>,h,l

" 可以在buffer的任何地方使用鼠标（类似office中在工作区双击鼠标定位）

set mouse=a

set selection=exclusive

set selectmode=mouse,key

" 通过使用: commands命令，告诉我们文件的哪一行被改变过

set report=0

" 在被分割的窗口间显示空白，便于阅读

set fillchars=vert:\ ,stl:\ ,stlnc:\

" 高亮显示匹配的括号

set showmatch

" 匹配括号高亮的时间（单位是十分之一秒）

set matchtime=1

" 光标移动到buffer的顶部和底部时保持3行距离

set scrolloff=3

" 为C程序提供自动缩进

set smartindent

" 高亮显示普通txt文件（需要txt.vim脚本）

au BufRead,BufNewFile *  setfiletype txt

"自动补全

:inoremap ( ()<ESC>i

:inoremap ) <c-r>=ClosePair(')')<CR>

:inoremap { {<CR>}<ESC>O

:inoremap } <c-r>=ClosePair('}')<CR>

:inoremap [ []<ESC>i

:inoremap ] <c-r>=ClosePair(']')<CR>

:inoremap " ""<ESC>i

:inoremap ' ''<ESC>i

function! ClosePair(char)

    if getline('.')[col('.') - 1] == a:char

        return "\<Right>"

    else

        return a:char

    endif

endfunction

filetype plugin indent on

"打开文件类型检测, 加了这句才可以用智能补全

set completeopt=longest,menu

"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
" CTags的设定
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

let Tlist_Sort_Type = "name"    " 按照名称排序

let Tlist_Use_Right_Window = 1  " 在右侧显示窗口

let Tlist_Compart_Format = 1    " 压缩方式

let Tlist_Exist_OnlyWindow = 1  " 如果只有一个buffer，kill窗口也kill掉buffer

let Tlist_File_Fold_Auto_Close = 0  " 不要关闭其他文件的tags

let Tlist_Enable_Fold_Column = 0    " 不要显示折叠树

autocmd FileType java set tags+=D:\tools\java\tags

"autocmd FileType h,cpp,cc,c set tags+=D:\tools\cpp\tags

"let Tlist_Show_One_File=1            "不同时显示多个文件的tag，只显示当前文件的

"设置tags

set tags=tags

"set autochdir



""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
"其他东东
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

"默认打开Taglist

let Tlist_Auto_Open=1

""""""""""""""""""""""""""""""

" Tag list (ctags)

""""""""""""""""""""""""""""""""

let Tlist_Ctags_Cmd = '/usr/bin/ctags'

let Tlist_Show_One_File = 1 "不同时显示多个文件的tag，只显示当前文件的

let Tlist_Exit_OnlyWindow = 1 "如果taglist窗口是最后一个窗口，则退出vim

let Tlist_Use_Right_Window = 1 "在右侧窗口中显示taglist窗口

" minibufexpl插件的一般设置

let g:miniBufExplMapWindowNavVim = 1

let g:miniBufExplMapWindowNavArrows = 1

let g:miniBufExplMapCTabSwitchBufs = 1
let g:miniBufExplModSelTarget = 1
```
