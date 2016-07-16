title: arch
author: N.C.Yi
tags: []
categories: []
date: 2016-07-16 16:15:00
---

## pacman
pacman有两个配置文件:`/etc/pacman.conf`和`/etc/pacman.d/mirrorlist`。前者指定根据什么来更新，通常是用后者。mirrorlist文件保存了一系列镜像地址，在进行`pacman -Syu`时根据选择的镜像来更新。

## 图形界面的安装
要使用图形界面，可以安装集成好的桌面环境(DE)如Gnome、KDE等，也可以自由搭配安装桌面环境中的各个组件。最重要的组件是window manager（wm）和display server。display server似乎只有Xorg和Wayland，大部分使用的是Xorg。我现在使用的wm是awesome。

安装Xorg
> pacman -S xorg-server

安装awesome
> pacman -S awesome

display manager常常被DE用来启动桌面环境以及管理用户登录，我们自行安装的wm也需要dm，不过也可以直接startx。

startx是xinit的前段或包装，主要启动x server和执行DE、WM（每个都是一个client）等等。`
~/.xserverrc`是startx用来启动x server的脚本。`/etc/X11/xinit/xserverrc`是其系统版本。系统版本的图形界面会在一个新的virtual terminal启动，为了限制它在启动它的vt中，需要
> #!/bin/sh 
> exec /usr/bin/Xorg -nolisten tcp "$@" vt$XDG_VTNR

`.xinitrc`用来指定xinit启动的client。对应系统版本为` /etc/X11/xinit/xinitrc`。要自定义这个文件，首先要copy系统版本再其基础上修改，因为需要执行其他一系列脚本。自定义结果为
```
...

if [ -d /etc/X11/xinit/xinitrc.d ] ; then
    for f in /etc/X11/xinit/xinitrc.d/?*.sh ; do
        [ -x "$f" ] && . "$f"
    done
    unset f
fi

# twm &
# xclock -geometry 50x50-1+1 &
# xterm -geometry 80x50+494+51 &
# xterm -geometry 80x20+494-0 &
# exec xterm -geometry 80x66+0+0 -name login

## some applications that should be run in the background
#xscreensaver &
#xsetroot -cursor_name left_ptr &

exec awesome
```
exec语句是最后一句，因为exec会进行进程替换。其他client应该fork或在后台执行。

需要注意的是，awesome默认使用的是xterm，arch刚刚安装完是没有xterm的，会出现打开terminal没有反应。安装xterm即可。

要启动图形界面，直接startx。awesome有退出选项。

要在登录时自动执行X，使用bash在`~/.bash_profile`中加入
>[[ -z $DISPLAY && $XDG_VTNR -le 3 ]] && exec startx

这样vt1~vt3都可以自动执行X了。

还有在使用中切换DE/WM，暂时没用到。

## 网络相关
DNS服务器配置文件是`/etc/resolv.conf`，host文件是`/etc/hosts`，dhcp配置在`/etc/dhcpcd.conf`（默认使用duid，如果无法联网可尝试clientid，血的教训/(ㄒoㄒ)/~~）。还有很多在`/etc/netctl`中，比如静态ip等等。