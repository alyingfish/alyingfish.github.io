---
title: Arch Linux 桌面环境与应用安装配置
date: 2025-07-19 22:12:45
categories:
  - [系统安装与配置, Arch Linux]
tags:
  - Archlinux
index_img:
banner_img:
---

> 本文为[安装 Arch Linux 到移动硬盘上](./arch-install/)的后续应用安装、配置说明。
> 也就是：桌面环境/窗口管理器、输入法、Shell、终端、编辑器、浏览器。

<!-- more -->

## 安装桌面环境或窗口管理器

流行的桌面环境有 [GNOME](https://wiki.archlinux.org/title/GNOME) 和 [KDE](https://wiki.archlinux.org/title/KDE)。

除此之外，还可以了解一下各种[窗口管理器](https://wiki.archlinux.org/title/Window_manager)。

平铺式窗口管理器：

- 窗口之间不会重叠，会自动分割并填充屏幕；
- 没有任务栏，全部使用虚拟桌面来管理/布局不同的窗口;
- 支持全键盘操作。

对于平铺式窗口管理器，推荐 [Hyprland](https://hypr.land/)。

### Hyprland

之所以选择 [Hyprland](https://hypr.land/)，最重要的就是其**华丽的动画和视觉效果**，然后就是**丰富强大可配置选项**以及**Wayland 原生支持**。

Hyprland 并非开箱即用，需要大量的配置，因此可以先使用大佬们的 [dotfiles](https://wiki.hypr.land/Getting-Started/Preconfigured-setups/)。

笔者使用过的有 [HyDE](https://github.com/HyDE-Project/HyDE)和 [end4's dotfiles](https://github.com/end-4/dots-hyprland)
安装这些 dotfiles 需要科学上网，否则可能导致部分资源因被墙而下载失败。

#### [HyDE](https://github.com/HyDE-Project/HyDE)

真正的开箱即用，从 GRUB、SDDM 的主题的安装到 Shell、终端、浏览器的设定。甚至对于 NVIDIA 显卡用户，直接把显卡驱动和各种针对 NVIDIA 的各种设置都直接安装并设定好了。当然输入法还是要自己装。

丰富的主题，内置就有七八种主题，可以一键灵活切换。

使用 Waybar 因此组件的外观有些简陋。

#### [end4's dotfiles](https://github.com/end-4/dots-hyprland)

使用 Quick Shell，因此顶栏和其组件更美观，并且有 Dock 栏和左右侧边栏，以及虚拟桌面预览——实时的。

Quick Shell 版本正在积极开发中，因此会出现各种 bug

基于壁纸颜色自动设定主题颜色。

## 安装配置输入法

输入法和桌面环境/窗口管理器关联较大，这里的配置仅在 `Hyprland` 下测试过。

### 安装输入法相关包

```bash
sudo pacman -S fcitx5-im # 输入法基础包组
sudo pacman -S fcitx5-chinese-addons # 官方中文输入引擎
sudo pacman -S fcitx5-configtool # 输入法设置工具
sudo pacman -S fcitx5-rime # 安装 rime 输入法
```

### 添加环境变量

创建 `~/.config/environment.d/im.conf`，并添加(详细信息见 [fcitx5 in wayland](https://fcitx-im.org/wiki/Using_Fcitx_5_on_Wayland#Chromium_.2F_Electron))：

```conf
XMODIFIERS=@im=fcitx
QT_IM_MODULE=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus
INPUT_METHOD=fcitx
```

`Hyprland` 则在 `~/.config/hypr/userprefs.conf` 中添加：

```conf
env = XMODIFIERS,@im=fcitx
env = QT_IM_MODULE,fcitx
env = SDL_IM_MODULE, fcitx
env = GLFW_IM_MODULE, ibus
env = INPUT_METHOD, fcitx
```

### 启动fcitx5，安装rime输入法

```bash
fcitx5 -d # 启动fcitx5
```

在终端中运行 `fcitx5-configtool`（或者右键任务栏的输入法图标，点击 configure），打开设置界面，取消勾选 `Only Show Current Language`，在上方搜索 Rime 并将其移动到左侧，点击 `Apply` 并退出即可。

此时，使用 `Ctrl`+`Space` 即可切换输入法并能够输入中文。

第一次使用时，会自动生成 rime 配置文件，在 `~/.local/share/fcitx5/rime` 中。

### 仅设置 `RightShift` 切换中英文输入法

我习惯仅使用 `RightShift` 切换输入法，这样不影响 `Shift` 输入大写。

`RightShift` 的按键默认识别为 `LeftShift`，因此还需要将 `RightShift` 改键为 "RightShift"，改键推荐使用 [keyd](https://github.com/rvaiya/keyd)。

将 `RightShift` 设置为 `Trigger Input Method` 即可。

### 使用雾凇拼音词库

Rime 默认的中文输入是否简陋，这里推荐 [雾凇拼音](https://github.com/iDvel/rime-ice)。

```bash
yay -S rime-ice # 安装雾凇拼音输入方案

```

创建 `~/.local/share/fcitx5/rime/default.custom.yaml` 并添加：

```yaml
patch:
  # 仅使用「雾凇拼音」的默认配置，配置此行即可
  __include: rime_ice_suggestion:/
  # 以下根据自己所需自行定义
  __patch:
    menu/page_size: 7 #候选词个数
    schema_list: # 不使用小鹤双拼去掉下两行
      - schema: double_pinyin_flypy

  # 禁止`shift`切换Rime输入法中英文，仅通过切换输入法切换中英文，因为Rime输入法的中英文切换是全局的，而输入法是可以局限在应用的（一个应用对应一个输入法，对应是否中英文）
  ascii_composer/switch_key/Shift_L: noop
  ascii_composer/switch_key/Shift_R: noop
```

使用小鹤双拼的话，还需要添加小鹤双拼补丁，原因见：<https://github.com/iDvel/rime-ice/tree/main/others/%E5%8F%8C%E6%8B%BC%E8%A1%A5%E4%B8%81%E7%A4%BA%E4%BE%8B>

```double_pinyin_flypy.custom.yaml
# 这是小鹤的补丁，其他双拼将文件名前面那部分改成对应方案 ID 就可以了
patch:
  # （按需选择）清空 preedit_format 中的内容，输入时显示双拼编码
  translator/preedit_format: []
```

之后重新启动 fcitx5（右键图标点击 `restart`)，`Ctrl+Space` 即可输入中文。

记得设置自动启动，Hyprland 在 `~/.config/hypr/hyprland.conf` 中设置:

```conf
exec-once = fcitx5 --replace -d
```

### 输入法美化

这里使用 fcitx5 的 [FluentDark](https://github.com/Reverier-Xu/Fluent-fcitx5) 皮肤，透明黑色。

![FluentDark](arch-apps/fcitx-fluent-dark.png "FluentDark 皮肤示例")

安装：

```bash
# Dark theme
yay -S fcitx5-skin-fluentdark-git
# Light theme
yay -S fcitx5-skin-fluentlight-git
```

之后进入fcitx5-configtool，在 `Addons`-`UI`-`Classic User Interface` 中，在 `Theme` 和 `Dark Theme` 中下拉选中自己想要的主题即可。

## GRUB

### GRUB 主题

这个仓库集成了许多好看的 GRUB 主题：<https://github.com/Jacksaur/Gorgeous-GRUB>

这里推荐 [Elegant-grub2-themes](https://github.com/vinceliuice/Elegant-grub2-themes)

### Elegant-grub2-themes

从 `1f7907` 后，原有的 DejaVuSans 字体被替换成了 Grub 默认的 unifont 字体。为此你可能需要手动替换之前的字体，或使用在这之前的版本。

## SDDM

### SDDM 主题

SDDM 主题可按自己的喜好下载。这里推荐 [sddm-astronaut-theme](https://github.com/Keyitdev/sddm-astronaut-theme)，里面集成许多主题，可以自行选择。

主题目录基本都是 `/usr/share/sddm/themes/<theme-name>`。

设置方法见对应主题的说明。

### 设置 SDDM 触摸板轻触为点击

SDDM 默认运行在 Xorg 上，因此要修改 X11 的设置：

添加文件 `/etc/X11/xorg.conf.d/20-touchpad.conf` 并填入下面的内容：

```conf
Section "InputClass"
        Identifier "libinput touchpad catchall"
        MatchIsTouchpad "on"
        MatchDevicePath "/dev/input/event*"
        Driver "libinput"

        Option "Tapping" "on"
        Option "NaturalScrolling" "on"
        Option "MiddleEmulation" "on"
        Option "DisableWhileTyping" "on"
EndSection
```

## 动态壁纸

这里使用 [mpvpaper](https://github.com/GhostNaN/mpvpaper)，可以将视频作为桌面，并支持mpv的设置。

### 安装

```bash
sudo pacman -S mpvpaper
```

### 设置自动启动

在`~/.config/hypr/userprefs.conf`中添加：

```conf
exec-once = mpvpaper -f -n 7200 -o "input-ipc-server=/tmp/mpv-socket --shuffle --loop --loop-playlist --panscan=1.0 --osd-level=0" "*" /home/Videos/Wallpapers
```

参数解释:

mpvpaper详细参数意义见`man mpvpaper`，mpv详细设置见<https://mpv.io/manual/master>

- `-f` -- fork mpvpaper从而可以关闭终端
- `-n 7200` -- 幻灯片模式每2小时（7200秒）播放播放列表中的下一个视频，需配合`--loop`, `--loop-playlist`使用
- `-o` -- mpvpaper传递参数给mpv
  - `input-ipc-server=/tmp/mpv-socket` -- 提供mpvpaper的控制接口
  - `shuffle` -- 启动时打乱播放列表
  - `loop` -- 循环播放视频
  - `loop-playlist` -- 循环播放列表
  - `panscan=1.0` -- 拉伸以填充整个屏幕（不留黑边）
  - `osd-level=0` -- 去除所有mpv渲染在视频上的OSD信息，避免禁音mpv时显示"Mute: yes"
- `"*"` -- 显示在所有屏幕上
- `/home/Video/Wallpapers` -- # 播放的可以是视频文件或者包含视频文件的文件夹

参数过多，可以设置mpv配置文件，再引用配置文件`~/.config/mpv/mpv.conf`，详细见<https://mpv.io/manual/master/#configuration-files>：

```conf
[mpvpaper]
profile-desc="profile for mpvpaper"
vo=gpu-next
gpu-api=auto
hwdec=auto-safe
profile=fast
input-ipc-server=/tmp/mpv-socket
shuffle
loop
loop-playlist
panscan=1.0
osd-level=0
```

之后自动启动便可写为：

```conf
exec-once = mpvpaper -f -n 7200 -o "profile=mpvpaper" "*" /home/Videos/Wallpapers
```

### 设置mpvpaper桌面图层排序

此外，为避免视频桌面被其他桌面程序覆盖(HyDE中是swww-daemon)，可以设置mpvpaper的layer优先度最低，或者直接不启动其他桌面程序：

```conf
layerrule = order -1, mpvpaper
```

### 自动暂停、禁音

mpvpaper提供`--auto-pause`和`--auto-stop`的参数，但在hyprland中没有效果，因此自己写了两个脚本：

- [auto_pause_mute_mpvpaper.sh](https://gist.github.com/coinhere/b97695322f9079a2178bb55120f2a795)，作用是：
  - 工作区窗口数量由0变为1时静音，由1变为0时取消静音
  - 窗口全屏时（非最大化）暂停，退出全屏时取消暂停
  - 切换工作区后，根据工作区窗口数量和是否全屏选择禁音、暂停mpvpaper与否
- [mpvpaper.sh](https://gist.github.com/coinhere/b97695322f9079a2178bb55120f2a795#file-mpvpaper-sh)，作用是：
  - 如果以及有mpvpaper.sh进程则直接退出
  - 启动mpvpaper和auto_pause_mute_mpvpaper.sh
  - 退出时关闭mpvpaper和auto_pause_mute_mpvpaper.sh
  - 每隔一秒检测系统是否有其他音频输出，有则将mpvpaper音量降至零

需要开启mpvpaper控制接口，并安装`socat`

自动启动修改为：

```conf
exec-once = $HOME/.config/hypr/scripts/mpvpaper.sh
```

### 添加控制壁纸快捷键

在`~/.config/hypr/keybindings.conf`：

需要开启mpvpaper控制接口，并安装`socat`

```conf
# mpv-paper
# 切换mpvpaper运行状态，关闭/启动mpvpaper及相关进程
bindd = $mainMod, F4, $d toggle mpvpaper, exec, pkill -x mpvpaper.sh >/dev/null || ~/.config/hypr/scripts/mpvpaper.sh
bindd = $mainMod, F5, $d toggle mpvpaper voice, exec, echo 'cycle mute' | socat - /tmp/mpv-socket # 静音/取消静音
bindd = $mainMod, F6, $d mpvpaper play prev, exec, echo 'playlist-prev' | socat - /tmp/mpv-socket # 播放上一个
bindd = $mainMod, F7, $d toggle mpvpaper play, exec, echo 'cycle pause' | socat - /tmp/mpv-socket # 暂停/取消暂停
bindd = $mainMod, F8, $d mpvpaper play next, exec, echo 'playlist-next' | socat - /tmp/mpv-socket # 播放下一个
```

### 自动退出、启动动态壁纸

hyprlock锁屏时，动态壁纸还会运行，因此写了个脚本锁屏时退出mpvpaper，解锁时再打开mpvpaper，这里使用上面脚本启动mpvpaper：

```bash
#!/bin/bash

lock() {
  # avoid starting multiple hyprlock instances.
  if ! pidof hyprlock >/dev/null; then
    # quit mpvpaper script
    pkill -x mpvpaper.sh
    # pause music
    # playerctl pause
    # when unlock, restart mpvpaper
    lockscreen.sh && lock_hook
  fi
}

lock_hook() {
  # run mpvpaper again
  # playerctl play
  ~/.config/hypr/scripts/mpvpaper.sh
}

lock
```

## 改键

需求：

- `CapsLock`单击为`Escape`
- `CapsLock` + `f,b,p,n,a,e,u,d` = `right`, `left`, `up`, `down`, `home`, `end`, `pageup`, `pagedown`
- `CapsLock` + `h,j,k,l` = `left`, `down`, `up`, `right`
- `Escape`为`CapsLock`
- 右`Ctrl`键与右`Alt`键互换

这里选用[keyd](https://github.com/rvaiya/keyd)来改键，如果只是单纯交换键位可以使用Hyprland自带的[改键配置]()，

安装并启用keyd：

```bash
sudo pacman -S keyd
sudo systemctl enable --now keyd
```

要查看各个键位的名称，运行并按下要查看的键位：

```bash
sudo keyd monitor
```

添加配置文件`/etc/keyd/default.conf`，如下所示：

```conf
[ids]

*

[main]

# Maps capslock to escape when pressed and control when held.
capslock = overload(capslock_layer, esc)

# Remaps the escape key to capslock
esc = capslock
rightalt = rightcontrol
rightshift = rightshift

[capslock_layer]
f = right
b = left
p = up
n = down
a = home
e = end
u = pageup
d = pagedown
h = left
j = down
k = up
l = right
space = backspace
backspace = delete
```

运行以下命令来重载配置：

```bash
sudo keyd reload
```

## 其他推荐的应用

- 科学上网。请查阅[简明指南-透明代理](https://arch.icekylin.online/guide/rookie/transparent.html)
- [fish shell](https://fishshell.com/)——用户友好的 Shell，自带许多有用功能，比需要加载一堆插件的 zsh 启动要快的多。不如的是，fish shell 不兼容 POSIX
- yazi
- [neovim](https://neovim.io/)——完全兼容 vim，同时插件生态极其丰富。觉得配置麻烦的可以使用 [lazyvim](http://www.lazyvim.org/installation)，完全可以说是一个 IDE 了。
- [neovide](https://neovide.dev/)——搭配 neovim，一个Neovim的图形用户界面。有更好的视觉及输入体验，动画丝滑无比。
- [vscode](https://wiki.archlinux.org/title/Visual_Studio_Code)——无须多言。
- [openRGB](https://openrgb.org/)——光污染必备
- [btop](https://github.com/aristocratos/btop)——一个系统资源监视器，类似任务管理器
- [pyprland](https://hyprland-community.github.io/pyprland/)——为 hyprland 用户提供了一下有用的插件，主要用到的是 `scratchpads`

## 个性化配置

关于 hyprland、终端、Shell、编辑器和各种软件的具体配置文件，可以查看我的 [dotfiles](https://github.com/alyingfish/dotfiles)
