## YouCompleteMe插件

### YouCompleteMe命令

#### YCM全局命令

- `:YcmRestartServer`

  重新启动`ycmd completion`服务器，当该服务器由于某些原因被停止，可以使用该命令

- `:YcmFocusCompileAndDiagnostics`

  强制YCM立刻重新编译你的文件并展示并显示任何遇到的新诊断。需要注意的是：用这个命令来重新编译，会阻塞Vim GUI一段时间

- `:YcmDiags`

  如果在你的文件中检测到错误或警告，则将错误或警告放入到vim的`locationlist`中并将其打开。如果一个错误或警告能够用命令`:YcmCompleter FixIt`修复，那么`FixIt available`会被追加到这个错误或警告文本的后面。需要注意的是，`FixIt available`的缺失并不是严格意义上的修复——它是不可用的，应为并不是所有的补全程序都能提供这个提示。可以使用`g:ycm_open_locallist_on_ycm_diags`这个选项来阻止这个`location list`打开，但是仍然会在当中填充新的诊断数据

- `:YcmShowDetailDiagnostic`

  显示所有的诊断数据

- `:YcmDebugInfo`

  打印当前文件的各种调试信息

- `:YcmToggleLogs`

  这个命令为当前文件类型提交由YCM,ycmd服务器，和语义引擎服务器创建的日志文件列表。这些日志文件能够在编辑器中通过确定对应的数字或用鼠标点击它来打开。此外，这个命令还能够将日志文件的名称作为参数，也就是说，用该命令打开指定的日志文件

- `:YcmCompleter`

  这条命令在YCM中提供大量的类IDE特性，如语义GoTo, 类型信息，修复信息，重构。该命令接受一个范围，该范围可以通过Vim的一种可视化模式(参见:h visual-use)中的选择来指定，也可以在命令行上指定。

### YcmCompleter 子命令

已被调用的子命令会自动的被路由到当前激活的语义补全程序，因此，如果当前活动文件是一个python文件，`:YcmCompleter GoToDefinition`将在Python语义补全程序中调用`GoToDefinition`子命令，如果是一个C族语言文件，将在clang补全程序程序中调用`GoToDefinition`子命令。

#### GoTo 命令

- `GoToInclude` 查找当前文件的头文件并跳转到头文件的位置
  - 支持的文件类型：`c,cpp,objc,objcpp,cuda`
  
- `GoToDeclaration` 查找游标所在的标识符并跳转到它的声明
  - 支持的文件类型: `c,cpp,objc,objcpp,cuda,cs,go` `java,javascript,python,rust,typescript`
  
- `GoToDefinition` 查找光标所在的标识符并跳转到它的定义出
  - 注意： 对于C族语言来说，当标识符的定义是在当前的转换单元中，只有在确定的环境中才能够有效。一个转换单元由当前编辑的所有文件以及用`#include`（直接或间接）包含的所有文件组成。
  - 支持的文件类型：`c,cpp,objc,objcpp,cuda,cs,go` `java,javascript,python,rust,typescript`
  
- `GoTo` 这个命令尝试执行他所能执行的“最合理”的GoTo操作。当前，意味着他将尝试去查找光标所在标识符并跳转到它可能拥有的所有定义；如果这个定义在当前转换单元中是不可访问的，则跳转到标识符的声明处。对于C族语言而言，它首先查找当前头文件并跳转，对于Csharp而言，实现也会被考虑和优先处理。
  - 支持的文件类型：`c,cpp,objc,objcpp,cuda,cs,go` `java,javascript,python,rust,typescript`

- `GoToImprecise` 这个命令与`GoTo`命令类似，除了在AST中查找到结点之前，不能够用`libclang`重编译这个文件。当您编辑需要长时间编译的文件，但您知道自上次解析以来没有进行任何会导致错误跳转的更改时，这可能非常有用。当您只是浏览代码库时，这个命令可以节省相当多的延迟时间。

  >  ## 这个命令用正确性换取速度!

- `GoToSymbol <symbol query>` 查找匹配到指定字符的所有标识符的定义。注意，这没有使用任何类型的智能/模糊匹配。但是，可以使用交互式符号搜索。
  - 支持的文件类型：`c,cpp,objc,objcpp,cuda,cs,go` `java,javascript,python,rust,typescript`
  
- `GoToImplementation` 它不支持C族语言

- `GoToImplementationElseDeclaration` 仅Csharp支持

- `GoToType` 仅`go,java,javascript,typescript`支持

- `GoToDucumentOutline` 在当前文件中提供一个标识符列表，存放在在quickfix list列表中。

#### 语义信息命令

这些命令查找代码的静态信息是非常有用的，例如变量的类型，显示声明和文档字符串

- `GetType` 回显游标下的变量或方法的类型，不同的地方是派生类型。
  - 支持的文件类型：`c,cpp,objc,objcpp,cuda,cs,go` `java,javascript,python,rust,typescript`
- `GetTypeImprecise` 与`GetType`命令类似
- `GetParent`  回显当前游标所在标识符的上一级信息
- `GetDoc` 显示预览窗口，显示关于光标下标识符的快速信息。依赖的文件类型如下所示：
  - 类型或声明的标识符
  - Doxygen/javadoc 注释
  - Python docstrings
  - etc
- `GetDocImprecise` 与`GetDoc`类似

#### 重构命令

这些命令对源代码进行更改，以便执行重构或代码纠正。YouCompleteMe不执行任何不能撤消的操作，并且从不保存或写入文件到磁盘。

- `FixIt` 在可能的情况下，尝试对缓冲区进行更改以纠正当前行上的诊断。当有多个建议可用时(例如有多种方法来解决给定的警告，或报告当前行的多个诊断)，将显示选项，并可以选择一个。提供诊断的补全器也可能对源进行微小的修改，以纠正诊断结果。示例包括语法错误，如缺少结尾分号、虚假字符，或语义引擎可以确定地建议更正的其他错误。如果对当前行没有可用的fix-it，或者对当前行没有诊断，则该命令对当前缓冲区没有影响。如果进行了任何修改，对缓冲区所做的修改的数量将被回显，用户可以使用编辑器的undo命令来恢复。当诊断可用，并且`g:ycm_echo_current_diagnostic`被设置为1时，当补全器能够添加这个指示时，文本(FixIt)被添加到回显的诊断。文本(可用FixIt)还被附加到`:ycmdaigs`命令的输出中的诊断文本中，用于任何具有可用fix-its的诊断(补全程序可以提供此指示)。

#### 多文件重构

当一个Refactor或FixIt命令触及多个文件时，YouCompleteMe尝试将这些修改应用到当前选项卡中任何现有的打开的、可见的缓冲区。如果找不到这样的缓冲区，YouCompleteMe在当前窗口的顶部以一个新的小的水平分割打开文件，应用更改，然后隐藏窗口。注意:缓冲区保持打开，必须手动保存。在执行此操作之前会打开一个确认对话框，以提醒您即将发生此操作。

修改完成后，`quickfix list`(参见:help quickfix)将填充所有修改的位置。这可以用来检查通过使用:copen所做的所有自动更改。通常，使用CTRL-W 组合在新的拆分中打开所选文件。可以使用YcmQuickFixOpened自动命令来定制快速修复窗口的打开方式。

缓冲区不会自动保存。也就是说，在从快速修复列表查看更改之后，必须手动保存修改的缓冲区。可以使用Vim强大的撤销功能撤消更改(参见:帮助撤销)。注意，Vim的撤消是针对每个缓冲区的，因此要撤消所有更改，必须在每个修改的缓冲区中分别应用撤消命令。

注意:在应用修改时，Vim可能会找到已经打开并具有交换文件的文件。如果在任何此类提示中选择Abort或Quit，则该命令将被终止。这使得Refactor操作只完成了一部分，必须使用Vim的撤消特性手动纠正。在这种情况下，没有填充快速修复列表。Inspect:buffers或等价的(参见:help buffers)以查看被命令打开的缓冲区。

- `Format`该命令根据Vim选项的shiftwidth和expandtab的值格式化整个缓冲区或其中的一部分(分别参见:h 'sw'和:h et)。要格式化文档的特定部分，您可以在Vim的一种可视化模式中选择它(参见:h visual-use)并运行命令或直接在命令行上输入范围，例如:2,5ycmcompleter format来格式化它从第2行到第5行。

### YouCompleteMe函数

- `youcompleteme#GetErrorCount`函数

  获取YCM诊断错误信息数，如果没有错误信息，则返回0

  ```c
  call youcompleteme#GetErrorCount()
  ```

- `youcompleteme#GetWarningCount`函数

  获取YCM诊断警告信息数，如果没有警告信息，则返回0

  ```shell
  call youcompleteme#GetWarningCount()
  ```

- `youcompleteme#GetCommandResponse(...)`函数

  运行补全子命令并以字符串形式返回结果。这可能很有用，例如在一个弹出窗口中显示GetGoc输出

  ```shell
  let s:ycm_hover_popup = -1
  function s:Hover()
    let response = youcompleteme#GetCommandResponse('GetDoc')
    if response == ''
      return
    endif
    
    call popup_hid(s:ycm_hover_popup)
    let s:ycm_hover_popup = popup_atcursor( balloon_split( response ), {} )
  endfunction
  
  " CursorHold triggers in normal mode after a delay
  autocmd CursorHold * call s:Hover()
  
  " or,if you prefer, a mapping
  nnoremap <silent><leader>D :call <SID>Hover()<CR>
  ```

  




## Ctags

### Ctags的安装

```shell
$sudo apt install ctags
```

### Ctags的使用

#### 第一、生成索引文件

```shell
$ctags -R --c++-kinds=+px --fields=+iaS --extra=+q
# --c++-kinds=+px为记录C++文件中的函数声明和各种外部和向前声明 
# --fields为ctags要求描述的信息
# +iaS
# i 标识如果有继承，则标识父类
# a 如果元素是类成员，要标明其调用权限
# S 如果是函数，则标识函数的signature
# --extra=+q
```

#### 第二、加载tags索引文件

在vim的命令行模式下输入如下命令

```shell
:set tags=path/to/tags
:set autochdir
```

#### 第三、跳转

- `<C-]>` 跳转到声明处或定义处
- `<C-t>` 跳回到原来的位置
- `:ts` 显示tags list
- `:tp` 上一个tag
- `:tn` 下一个tag



## cscope

### 安装cscope

```shell
$sudo apt install cscope
```

### 使用cscope

#### 第一、创建cscope数据库

```shell
$cscope -Rbq .
```

#### 第二、加载cscope数据库

##### 自动加载cscope数据库

在.vimrc中编写自动脚本，如下所示：

```sh
if has("cscope") " 查看是否有cscope这个应用程序
  if filereable("cscope.put") " 是否能读取到cscope数据库
     cs add cscope.out " 如果能够读取到当前数据库，就加载数据库
  else " 在其他的地方搜索能够使用的数据库
    let cscope_file=findfile("cscope.out", ".;")
    let cscope_pre=mathchstr(cscope_file,".*/")
    if !empty(cscope_file) && filereadable(cscope_file)
       set nocsverb
       exe "cs add" cscope_file cscope_pre
       set csverb
    endif
  endif
endif
```

##### 手动加载cscope数据库

在vim的命令行模式下，输入如下命令

```sh
:cs add path/to/cscope.out
```

#### 第三、cscope常用的命令

```sh
：cs f s ---- 查找C语言符号，即查找函数名、宏、枚举值等出现的地方
：cs f g ---- 查找函数、宏、枚举等定义的位置，类似ctags所提供的功能
：cs f d ---- 查找本函数调用的函数
：cs f c ---- 查找调用本函数的函数
：cs f t ---- 查找指定的字符串
：cs f e ---- 查找egrep模式，相当于egrep功能，但查找速度快多了
：cs f f ---- 查找并打开文件，类似vim的find功能
：cs f i ---- 查找包含本文件的文
```

```sh
-R: 在生成索引文件时，搜索子目录树中的代码 
-b: 只生成索引文件，不进入cscope的界面 
-q: 生成cscope.in.out和cscope.po.out文件，加快cscope的索引速度 
-k: 在生成索引文件时，不搜索/usr/include目录 
-i: 如果保存文件列表的文件名不是cscope.files时，需要加此选项告诉cscope到哪儿去找源文件列表。可以使用”-“，表示由标准输入获得文件列表。 
-Idir: 在-I选项指出的目录中查找头文件 
-u: 扫描所有文件，重新生成交叉索引文件 
-C: 在搜索时忽略大小写 
-Ppath: 在以相对路径表示的文件前加上的path，这样，你不用切换到你数据库文件所在的目录也可以使用它了。 
```



## rainbow插件

## NERDCommenter插件

### 配置

```sh
" Create default mappings
let g:NERDCreateDefaultMappings = 1

" Add spaces after comment delimiters by default
let g:NERDSpaceDelims = 1

" Use compact syntax for prettified multi-line comments
let g:NERDCompactSexyComs = 1

" Align line-wise comment delimiters flush left instead of following code indentation
let g:NERDDefaultAlign = 'left'

" Set a language to use its alternate delimiters by default
let g:NERDAltDelims_java = 1

" Add your own custom formats or override the defaults
let g:NERDCustomDelimiters = { 'c': { 'left': '/**','right': '*/' } }

" Allow commenting and inverting empty lines (useful when commenting a region)
let g:NERDCommentEmptyLines = 1

" Enable trimming of trailing whitespace when uncommenting
let g:NERDTrimTrailingWhitespace = 1

" Enable NERDCommenterToggle to check all selected lines is commented or not 
let g:NERDToggleCheckAllLines = 1
```

### 默认快捷键

- `[count]<leader>cc`  等价于 在可视模式下使用`:NERDCommenterComment` 在可视模式下注释当前行或选中的文本

- `[count]<leader>cn` 等价于 `:NERDComenterNested` 和上面的命令类似，不过它强制嵌套，即注释依然可以添加注释

- `[count]<leader>c<sapce>` 等价于 `:NERDCommenterToggle` 切换所选行的注释状态。如果对最上面的选中行进行了注释，则所有选中的行都不进行注释，反之亦然。

- `[count]<leader>cm` 等价于 `:NERDCommenterMinimal`‎仅使用一组多部分分隔符注释给定的行。‎

- `[count]<leader>ci` 等价于 `:NERDCommenterInvert` ‎单独切换所选行的注释状态。‎

- `[count]<leader>cs` 等价于 `:NERDCommenterSexy`‎使用漂亮的块格式布局注释掉所选行。‎

- `[count]<leader>cy` 等价于 `:NERDCommenterYank` 与`cc`类似，不同之处在于，先删除已经注释的行

- `<leader>c$` 等价`:NERDCommenterToEOL` 从当前注释到最后一行

- `<lader>cA` 等价 `:NERDCommenterAppend` 将注释分隔符添加到行尾，并在它们之间进入插入模式。

- `:NERDCommenterInsert`在当前光标位置添加注释分隔符，并在两者之间插入。默认情况下禁用。

- `<leader>ca` 等价 `:NERDCommenterAltDelims` 切换到另一组分隔符。

- `[count]<leader>cl` 等价 `:NERDCommenterAliginLeft`

- `[count]<leader>cb` 等价于 `:NERDCommenterAlignBoth`

  cl和cb与cc类似，只不过注释的对齐方式不同而已。

- `[count]<leader>cu` 等价于 `:NERDCommenterUncomment` 取消所选行的注释

除了`:NERDComenterInsert`以外，其他的所有注释命令既可以在普通模式下使用也可以在可视模式下使用

`cc cn cl cb cy`都单行单行的注释，只是对其方式不同而已

`cm` 是块注释

`ci`切换注释的形式，如果是单行注释，则切换到块注释，否则反之

`cs`用美化的注释块进行注释

`c<space>` 如果光标所在行没有注释，而在它下面的部分行无论有没有注释，则所选中的所有行都进行注释；如果光标所在行有注释，反之。

以上所有的命令只有`<leader>`按下后才能才能开始按他们

`c$` 从当前行开始注释到最后一行

`ca` 切换注释符 

`cA`在行尾添加一个注释

## vim-terminal-help插件

### 快捷键使用方式

- `ALT` + `=`: toggle terminal below.
- `ALT` + `SHIFT` + `h`: move to the window on the left.
- `ALT` + `SHIFT` + `l`: move to the window on the right.
- `ALT` + `SHIFT` + `j`: move to the window below.
- `ALT` + `SHIFT` + `k`: move to the window above.
- `ALT` + `SHIFT` + `n`: move to the previous window.
- `ALT` + `-`: paste register 0 to terminal.
- `ALT` + `q`: switch to terminal normal mode.

Inside the terminal:

```
drop abc.txt
```

tell vim to open `abc.txt`

## vim-translator插件

#### `Translate`

```sh
:Translate[!] [--engines=ENGINES] [--target_lang=TARGET_LANG] [--source_lang=SOURCE_LANG] [your text]
```

```sh
:Translate                                  " translate the word under the cursor
:Translate --engines=google,youdao are you ok " translate text `are you ok` using google and youdao engines
:2Translate ...                             " translate line 2
:1,3Translate ...                           " translate line 1 to line 3
:'<,'>Translate ...                         " translate selected lines
```

#### `TranslateW`

```
:TranslateW[!] [--engines=ENGINES] [--target_lang=TARGET_LANG] [--source_lang=SOURCE_LANG] [your text]
```

Like `:Translate...`, but display the translation in a window

#### `TranslateR`

```
:TranslateR[!] [--engines=ENGINES] [--target_lang=TARGET_LANG] [--source_lang=SOURCE_LANG] [your text]
```

Like `:Translate...`, but replace the current text with the translation

#### `TranslateX`

```
:TranslateX[!] [--engines=ENGINES] [--target_lang=TARGET_LANG] [--source_lang=SOURCE_LANG] [your text]
```

#### `TranslateH`

```
:TranslateH
```

Export the translation history

#### `TranslateL`

```
:TranslateL
```

Display log messageranslate the text in the clipboard

### Highlight

Here are the default highlight links. To customize, use `hi` or `hi link`

```
" Text highlight of translator window
hi def link TranslatorQuery             Identifier
hi def link TranslatorDelimiter         Special
hi def link TranslatorExplain           Statement

" Background of translator window border
hi def link Translator                  Normal
hi def link TranslatorBorder            NormalFloat
```

## vim-bookmarks插件

| Action                                        | Shortcut     | Command                       |
| --------------------------------------------- | ------------ | ----------------------------- |
| Add/remove bookmark at current line           | `mm`         | `:BookmarkToggle`             |
| Add/edit/remove annotation at current line    | `mi`         | `:BookmarkAnnotate <TEXT>`    |
| Jump to next bookmark in buffer               | `mn`         | `:BookmarkNext`               |
| Jump to previous bookmark in buffer           | `mp`         | `:BookmarkPrev`               |
| Show all bookmarks (toggle)                   | `ma`         | `:BookmarkShowAll`            |
| Clear bookmarks in current buffer only        | `mc`         | `:BookmarkClear`              |
| Clear bookmarks in all buffers                | `mx`         | `:BookmarkClearAll`           |
| Move up bookmark at current line              | `[count]mkk` | `:BookmarkMoveUp [<COUNT>]`   |
| Move down bookmark at current line            | `[count]mjj` | `:BookmarkMoveDown [<COUNT>]` |
| Move bookmark at current line to another line | `[count]mg`  | `:BookmarkMoveToLine <LINE>`  |
| Save all bookmarks to a file                  |              | `:BookmarkSave <FILE_PATH>`   |
| Load bookmarks from a file                    |              | `:BookmarkLoad <FILE_PATH>`   |

