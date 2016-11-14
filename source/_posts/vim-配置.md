title: vim 配置
tags:
  - vim
categories:
  - Vim
date: 2016-11-04 17:53:00
---
### 在 MinGW 中改变字体
在 [how-do-you-configure-msyss-default-size-color-and-font](http://stackoverflow.com/questions/447824/how-do-you-configure-msyss-default-size-color-and-font) 中提到，最简单的方法就是在窗口上边框点击右键配置相关参数。 我使用的是 [mintty](https://mintty.github.io/)，支持这样的配置方法。更复杂的是 RXVT，可能需要更改相应配置文件。

### Statusline 的配置
statusline 即 vim 窗口的状态栏，在命令栏的上方编辑区的下方。可以通过一行配置

```vim
set statusline=%t[%{strlen(&fenc)?&fenc:'none'},%{&ff}]%h%m%r%y%=%c,%l/%L\ %P
```

注意，不能出现空格，如果想加入字面上的空格需要用反斜线转义。

也可以用多行带注释的方式

{% codeblock lang:vim %}
set statusline=%t       "tail of the filename
set statusline+=[%{strlen(&fenc)?&fenc:'none'}, "file encoding
set statusline+=%{&ff}] "file format
set statusline+=%h      "help file flag
set statusline+=%m      "modified flag
set statusline+=%r      "read only flag
set statusline+=%y      "filetype
set statusline+=%=      "left/right separator
set statusline+=%c,     "cursor column
set statusline+=%l/%L   "cursor line/total lines
set statusline+=\ %P    "percent through file
{% endcodeblock %}  

`%t` 和 `%f` 有区别，可以观察在 `%f` 下把焦点移到另一个 buffer 后发生什么变化。

[making-statuslines-that-own](http://got-ravings.blogspot.com/2008/08/vim-pr0n-making-statuslines-that-own.html) 详细说明了配置要点，包括 `%{...}` 自定义内容，`%-0{minwid}.{maxwid}{item}` 的完全语法，`%(...%)` 将多个东西括起来等等。


### Vimdiff

#### E97
vimdiff \( 包括 git mergetool、fugutive 的 :Gdiff \) 都需要可执行文件 diff 的支持。在 MinGW 中如果出现 E97，可参考 [解决 E97](http://www.xbatu.com/node/19) 中的做法，简而言之使用 `ln -s /usr/bin/diff /usr/share/vim/vim74diff` 将 vim74diff 指向系统 diff。

diff 输出内容的解读可以参考 [how to read diff](http://www.ruanyifeng.com/blog/2012/08/how_to_read_diff.html)。

### Git Merge
回顾 git remote 和 git fetch 等命令可以看 [git 远程操作](http://www.ruanyifeng.com/blog/2014/06/git_remote.html)。

如果在本地修改了一些配置文件，想更新软件主要版本，可以使用 git stash 来完成 merge 过程：
```
git stash
git fetch
git merge origin/master
git stash pop
```
这时候会有冲突，使用 git mergetool 或者在 vim 中使用 fugitive 插件的 `:Gdiff` 均可。关于 fugitive 的使用可以观看 [fugitive.vim](http://vimcasts.org/episodes/fugitive-vim-resolving-merge-conflicts-with-vimdiff/)视频介绍，git mergetool 的使用可以查看 [vimdiff](http://blog.binchen.org/posts/use-vimdiff-to-resolve-git-merge-conflicts-effectively.html)的说明。