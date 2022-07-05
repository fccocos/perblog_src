# vim 快捷键_Vim快捷键映射mapping

![a09eceab8df67eea52ead4df1ec12d1a.png](https://img-blog.csdnimg.cn/img_convert/a09eceab8df67eea52ead4df1ec12d1a.png)

使用一段时间[vim](https://so.csdn.net/so/search?q=vim&spm=1001.2101.3001.7020)后，多多少少会开始接触vim的快捷键映射（mapping），快捷键映射（mapping）能够帮助我们大幅度提高使用vim编辑文档的效率。希望通过这篇文章，大家可以比较清晰的理解vim快捷键映射（mapping）的基本工作机制，并能够定制自己的快捷键。

先看一个简单的例子。我们可以将下面的脚本输入到vim[命令行](https://so.csdn.net/so/search?q=命令行&spm=1001.2101.3001.7020)（command line）中，或写入`~/.vimrc`中（永久保存映射关系）。

```python
map <F2> :echo 'Current time is ' . strftime('%c')<CR>
```

通过执行以上的脚本之后，在vim中，每次在键盘上按`F2`时，就相当于通过键盘敲击接下来的内容`:echo 'Current time is ' . strftime('%c')<CR>`。

大家可以手动试一下，查看效果，敲击冒号`:`进入命令行（command line），`<CR>`表示回车键。

## 快捷键映射（mapping）命令结构

我们可以查看vim关于快捷键映射（mapping）自带的帮助文档，`:help map.txt`。不过会有一些枯燥难懂。

vim快捷键映射（mapping）命令结构如下：

```python
map-commands map-arguments {lhs} {rhs}
```

### map-commands

第一个部分`map-commands`定义了

- 快捷键映射作用于哪种工作模式（mode）
- 快捷键映射是否可以递归（recursive）
- 是执行添加映射，还是删除映射，还是获取映射列表

例如`nnoremap`

- `n-` normal mode一般模式
- `-nore-` no recursive 不递归
- `-map` 添加映射

更多可以查看帮助文档`:help map-commands`

### map-arguments

第二部分`map-arguments`特殊参数，紧接着`map-command`，可选。

例如`<buffer>`, 表示当前映射仅作用于当前buffer，添加该参数后的完整映射（map）命令如下

```xml
map <buffer> <F2> :echo 'Current time is ' . strftime('%c')<CR>
```

### {lhs} left-hand side

第三部分`{lhs}`表示快捷键，希望使用的按键。可以是单个按键也可以是多个。例如上面的`<F2>`。

### {rhs} right-hand side

第四部分`{rhs}`表示当快捷键`{lhs}`触发之后将被执行的内容：

- 可以是一个活多个键盘按键
- vim内置的命名
- 也可以是用vimscript编写的方法

### 快捷键映射命令综合小结

让我们先通过下面的例子做一个小结

```ruby
nnoremap <slient> ,<space> :nohlsearch<CR>
```

每当我们在一般模式（normal mode）中，按下`,<space>`(逗号和空格），等同于敲击冒号`:`进入如命令行，然后输入命令`nohlsearch`并按下回车键。

其中`<slient>`告诉vim不要把输入到命令行的内容显示出来。

`-nore-`表示，如果`,<space>`出现在其它快捷键命令的`{rhs} right-hand side`部分，不会再被当做快捷键递归触发。

## 快捷键映射命令Map Commands - :map-commands

前面已经提过了，快捷键映射命令的三个职责

1. 作用于哪个工作模式（mode）
2. 是否可以递归（recursive）
3. 执行添加，删除，还是获取列表操作

一般可以通过第一个字母判断对对应的工作模式（mode），帮助文档中有更详细的资料

```shel
:help map-overview
     COMMANDS                    MODES

:map   :noremap  :unmap     Normal, Visual, Select, Operator-pending
:nmap  :nnoremap :nunmap    Normal
:vmap  :vnoremap :vunmap    Visual and Select
:smap  :snoremap :sunmap    Select
:xmap  :xnoremap :xunmap    Visual
:omap  :onoremap :ounmap    Operator-pending
:map!  :noremap! :unmap!    Insert and Command-line
:imap  :inoremap :iunmap    Insert
:lmap  :lnoremap :lunmap    Insert, Command-line, Lang-Arg
:cmap  :cnoremap :cunmap    Command-line
:tmap  :tnoremap :tunmap    Terminal-Job
```

命令中是否包含`-nore-`来判断是否接受递归，如上面表格`COMMANDS`中第二列。

当存在`{lhs}`和`{rhs}`表示添加映射（mapping）关系 ，例如

```cpp
:map <F3>  o#include
```

包含`-un-` 时表示删除，如上面表格`COMMANDS`中第三列，例如

```xml
:unmap <F3>
```

## 特殊参数 Special Arguments - :map-argments

特殊参数包括`<buffer>`, `<nowait>`, `<silent>`, `<special>`, `<script>`, `<expr>`和`<unique>`，它们可以以任意次序放置。

### `<buffer>`

`<bufefr>`表示快捷键映射命令只作用于当前buffer，也可以说时本地buffer映射。

### `<nowait>`

`<nowait>`表示无需等待。比方说一个本地buffer映射是`,`和一个全局映射是`,a`，如下

```xml
map <buffer> , :echo 'local buffer mapping'<CR>



map ,a aGlobal Mapping<ESC>
```

这个时候当我们按下逗号`,`键，vim需要等一段时间或者按下第二个键，才能确定触发哪个快捷键。

或者我们可以使用`<nowait>`参数告诉vim无需等待， 直接触发已经匹配好的快捷键。当然那一个长一点的快捷键`,a`在当前buffer就无法再使用了。

```xml
map <buffer><nowait> , :echo 'local buffer mapping'<CR>
```

### `<silent>`

`<silent>`表示不在命令行（command line）中显示任何信息

### `<special>`

没看明白，原文如下

> ```
> <special>` Define a mapping with <> notation for special keys, even though the "<" flag may appear in 'cpoptions'. This is useful if the side effect of setting 'cpoptions' is not desired. Example: `:map <special> <F12> /Header<CR>
> ```

### `<script>`

<script> 表示仅递归脚本（script）或插件（plugin）内部以<SID>打头的映射。

例如在`~/.vimrc`中有如下代码，当我们按下`,dt`，触发执行`<SID>(FindTopic)dd`，

- 其中 `<SID>(FindTopic)`会被递归到`/Topic<CR>`，即，搜索`Topic`
- 但`dd` 不做递归，就是按两个`d`，即删除当前行

```xml
noremap <SID>(FindTopic) /Topic<CR>
noremap dd :echo 'dd'
noremap <script> ,dt <SID>(FindTopic)dd
```

### `<expr>`

`<expr>` 表示`{rhs}`是表达式，表达式的返回值，作为最后映射的捷键序列。

例如在`~/.vimrc`中，当按下`,a`是就相当于按下`abcdef`.

```swift
inoremap <expr> ,a InsertSth()
func InsertSth()
  return "abcdef"
endfunc
```

### `<unique>`

`<unique>` 用于避免已经存在的快捷键被覆盖

## 小结

希望以上的内容对大家有所帮助，可以搭建自己的快捷键映射。