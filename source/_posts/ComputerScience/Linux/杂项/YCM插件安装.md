1. 安装vundle插件

   ```shell
   $git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
   ```

2. 配置vundle插件

   ```shell
   set nocompatible "去除vi一致性，必须要
   filetype off "必须要
   " 设置包括vundle和初始化相关的runtime path
   set rtp+=~/.vim/bundle/Vundle.vim
   call vundle#begin()
   " 中间为要安装的插件
   call vundle#end()
   filetype plugin indent on
   
   " 设置Vundle的快捷键
   map <A-v>l :PluginList<CR> " 插件列表
   map <A-v>i :PluginInstall<CR> " 安装插件
   map <A-v>s :PluginSearch " 搜索插件
   map <A-v>c :PluginClean<CR> " 清除未使用插件
   " 可以使用:h vundle命令来查看更多的细节
   ```

3. 下载YouCompleteMe源码, 将其下载到`~/.vim/bundle`中

   ```shell
   $git clone https://gitee.com/HangbinZheng/YouCompleteMe.git
   ```

4. 进入到YouCompleteMe目录中，用python3执行install.py文件

   ```
   $python3 install.py
   ```

   如果出现子模块缺失的错误提示，就执行如下代码

   ```shell
   $git submodule update --init --recursive
   ```

   由于网络原因，可能有些模块无法加载，记得多试几次。

5. 继续执行如下在代码

   ```shell
   $python3 install.py --clangd-completer
   ```

   如果出现python版本不符合要求的错误提示，那么就必须下载符合要求的python版本.

   如下所示，为在ubuntu中安装指定版本的python，并可以手动切换python的不同版本

   ```shell
   $sudo apt update
   $sudo apt install software-properties-common
   $sudo add-apt-repository ppa:deadsnakes/ppa
   $sudo apt install python3.8 python3.8-dev # 安装指定版本的pythonx.x
   $ls /usr/bin/python* # 查看所有版本的python
   $update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.8 1 # 为python3.8添加软连接
   $update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.10 2 # 为ptyhon3.10添加软连接
   $update-alternatives --list python3 # 查看python3对应的python的版本链接
   $update-alternatives --config python3 # 切换python3版本
   
   ```

   然后继续执行如下命令
   ```shell
   $python3 install.py --clangd-completer
   ```

   如果出现需要先到第三方库中运行build.py的提示，则执行如下代码

   ```shell
   $python3 ./third_party/ycmd/build.py
   ```

   编译完后，回到YouCompleteMe目中，继续执行如下命令

   ```shell
   $python3 install.py --clangd-completer
   ```

6. 在.vimrc中加入YCM插件, 如下所示

   ```shell
   call vundle#begin()
   Plugin 'Valloric/YouCompleteMe'
   call vundle#end()
   ```

7. 配置在`~/.vimrc`中配置YCM插件

   ```shell
   let g:ycm_show_diagnostics_ui = 0
   let g:ycm_server_log_level = 'info'
   let g:ycm_min_num_identifier_candidate_chars = 2
   let g:ycm_collect_identifiers_from_comments_and_strings = 1
   let g:ycm_complete_in_strings=1
   let g:ycm_key_invoke_completion = ''
   let g:ycm_semantic_triggers = {
   \ ‘c,cpp,python,java,go,erlang,perl’: [‘re!\w{2}’],
   \ ‘cs,lua,javascript’: [‘re!\w{2}’],
   \ }
   ```

   

