# vim简单配置 

* vim配置文件 .vimrc

  - 系统配置：系统配置文件存放在vim的安装路径内，默认路径：/usr/share/vim/vimrc
  - 用户配置：用户配置文件.vimrc由用户自己创建，存放于用户根目录下

* .vimrc文件编写

  （1）设置配色

  

  ```
  colorscheme desert
  ```

  （2）语法高亮

  如果不设置，在编写RTL时，显示的文字都将是一个颜色

  ```
  syntax on
  filetype on
  au BufRead,BufNewFile *.sv set filetype=systemverilog
  au BufRead,BufNewFile *.v  set filetype=verilog
  ```

  > syntax on是语法高亮的意思；filetype on文件类型识别；au是autocmd的缩写，BufRead和BufNewFile是触发自动命令的事件。BufNewFile是创建一个新文件，BufRead是将文件读入一个新缓冲区时触发。如：au BufRead,BufNewFile *.sv set filetype=systemverilog的意思是，当检测到文件类型是.sv结尾的文件时，将符合systemverilog语法的地方将高亮显示

  （3）设置行号：在左边显示第几行

  ```
  set number
  ```

  （4）自动缩进：换行时，缩进量与上一行对齐

  ```
  set autoindent
  ```

  （5）空格替代tab缩进

  ```
  set ts=2
  set expandtab
  ```

  > 目的是为代码整齐美观，由于每个人设置的tab缩进量不同，所以当你的代码换到别人的设备中打开时，可能就没有对齐，比较乱。这个时候，就需要设置空格替代tab，因为空格大家都一样。set ts=2：ts是tabstop的缩写，按一次tab键显示的宽度；set expandtab：设置之后，会把一个 tab 字符替换成 tabstop 选项值对应的数值空格。上面就是把按一次tab键缩进两个空格

  （6）设置搜索\行高亮

  ```
  set hlsearch
  set cursorline
  set cursorcolumn
  ```

  > set hlsearch：在vim的一般模式下，搜索某字符，这个字符将在文中高亮显示；set cursorline：光标所在行高亮显示；set cursorcolumn：所在列高亮显示，这个设置很有用。

  

（7）设置vim的字体大小

```
set guifont=Monospace\ 16
```

>set guifont=Monospace\ 16：其中Monospace为字体名，16为字号，注意\和16之间有一个空格

（8）符号匹配

```
inoremap ( ()<Esc>i
inoremap { {}<Esc>i
inoremap [ []<Esc>i
inoremap " ""<Esc>i
```

>在i模式下，按下左边的符号，会自动输入左右两边的符号，很好用的设置。

（9）调用别名文件

我的别名文件创建在 ~/.vim/user/alias.vim，下面先看看我平常设置的别名文件里的内容：

```
iab al_ <ESC>:r ~/.vim/user/always.v<cr>
iab mo_ <ESC>:r ~/.vim/user/module.v<cr>
iab an_ <ESC>:r ~/.vim/user/annotation.v<cr>
iab fs_ <ESC>:r ~/.vim/user/fsdb.v<cr>
iab tb_ <ESC>:r ~/.vim/user/tb.v<cr>
iab cyl_ <ESC>:r ~/.vim/user/cyl.v<cr>

iab cl_ <ESC>:r ~/.vim/user/clk.v<cr>
iab rs_ <ESC>:r ~/.vim/user/rst.v<cr>

"ab   model
"iab  edit model
"<cr> new line
"
```

> 以第一条为例：iab al_ :r ~/.vim/user/always.v，在vim编辑器edit 模式下，输入“al_”,再按下“ESC”键，路径~/.vim/user/下的always.v文件里的内容将会被调用，这就大大避免了重复的工作。

