---
date: 2019-01-24
title: Master Linux 
---

## Install Manjaro Linux 

### Burn ISO to usb stick:

1. Download mankato - GNOME [here](https://manjaro.org/download/gnome/)

2. Check the hash and compare with the hash provided on the site:

```shell
shasum ./manjaro-gnome-18.0.2-stable-x86_64.iso
```

3. Burn the iso file to USB stick on Linux

```shell
diskutil list
sudo umount /dev/disk2s4
sudo dd if=/path/image.iso of=/dev/rdisk2s4 bs=1m
```

	On windows, use [rufus](https://rufus.ie/) with `dd` mode.

### Bios set

* Insert USB stick
* Restart pc with `F2` into bios，set `secure boot` to be disable，F10 save and exit
* Press `F12` select start using USB.

### Windows setting

```
In the Control Panel app

Click on System and Security
Click on Power Options
Click on Choose what power buttons do
a. Click on Change settings that are currently unavailable
b. Uncheck the option Turn on fast startup
Click on Save Changes
```

## Software install

Use `tweaks` to adjust dock icon location and size

```shell
sudo pacman -S yay

yay -S shadowsocks-qt5

yay -S google-chrome

yay -S typora

yay -S neofetch

yay -S xorg-xev

sudo pacman -S zsh zsh-completions
cat /etc/shells
chsh -s /bin/zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```



## Configuration

### Turn CapsLock into Control

```shell
xmodmap -pke > ~/.Xmodmap
```

Add in `~/.Xmodmap`：

```shell
clear lock
clear control
keycode 66 = Control_L
add control = Control_L Control_R
```

Activate and restar, add `my-xmodmap.desktop`  in `.config/autostart`:

```shell
[Desktop Entry]
Exec=sh -c "xmodmap ~/.Xmodmap"
Name=my-xmodmap
Comment=Apply my xmodmap
Type=Application
Icon=nautilus`
```

### Create workspace

```shell
mkdir ~/workspace
mkdir ~/workspace/Github/dasheng
mkdir ~/workspace/Github/other
```

Sync time

```shell
sudo pacman -S ntp
sudo timedatectl set-ntp true
```

### Configure terminal

```
Ctrl+W: close tab
Ctrl+T: new tab
Ctrl+N: new window
Alt+Q: close window
```




## Reference

* [Create a USB installer on Mac OS X and install Manjaro](https://forum.manjaro.org/t/create-a-usb-installer-on-mac-os-x-and-install-manjaro-on-dell-precision-5520/21392)
* [Wiki: Windows 10 - Manjaro - Dual-boot - Step by Step](https://forum.manjaro.org/t/wiki-windows-10-manjaro-dual-boot-step-by-step/52668)
* [Manjaro 安装后的一些配置](https://www.jianshu.com/p/447003e8e482)
* [Swap Alt with Ctrl (keep window switcher Alt-Tab)](https://forum.manjaro.org/t/solved-swap-alt-with-ctrl-keep-window-switcher-alt-tab/36091/2)
* [Archlinux: xmodmap](https://wiki.archlinux.org/index.php/xmodmap)
* [nome 3.28 removes the option to display desktop icons](https://bbs.archlinux.org/viewtopic.php?id=235633)
* [Setup oh-my-zsh in Manjaro](https://forum.manjaro.org/t/how-to-setup-oh-my-zsh-in-manjaro/34519)
* [Use 1password in Linux](https://news.ycombinator.com/item?id=16719336)