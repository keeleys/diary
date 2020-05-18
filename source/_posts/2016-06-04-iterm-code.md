---
title: mac下打造漂亮的终端编辑器
date: 2016-06-04 11:13:13
tags: mac
---

## mac下打造漂亮的终端编辑器.

![https://dn-tianjun.qbox.me/screenshot%202016-06-04%2011.11.30.png](https://dn-tianjun.qbox.me/screenshot%202016-06-04%2011.11.30.png)

<!-- more -->
### iterm2
> mac自带终端应用的加强版，功能很强大。

![效果图](https://dn-tianjun.qbox.me/screenshot%202016-06-04%2010.56.16.png)

1. 安装步骤
`brew install Caskroom/cask/iterm2` 或者 down [https://www.iterm2.com/downloads.html](https://www.iterm2.com/downloads.html)

### oh-my-zsh & zsh-theme
1. 官网 [http://ohmyz.sh/](http://ohmyz.sh/)
2. 安装,mac自带了zsh模式 ，直接安装ohmyz就可以了，linux需要先安装zsh
```
$ sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

3. 主题浏览[https://github.com/robbyrussell/oh-my-zsh/wiki/Themes](https://github.com/robbyrussell/oh-my-zsh/wiki/Themes)

4. 安装主题
打开你的`~/.zshrc`文件,然后修改`ZSH_THEME=robbyrussell`,将`ZSH_THEME`设置成你喜欢的主题名字

### iterm2主题

1. 下载开源主题[https://github.com/chriskempson/tomorrow-theme](https://github.com/chriskempson/tomorrow-theme)

2. clone到本地然后配置到iterm2里面
```
$git clone git@github.com:chriskempson/tomorrow-theme.git
```
3. go to itemr2 ,Preferences/Colors/Color Presets and chose the theme

### vim主题

> 你的vim命令真的使用到了vim吗，查看你的电脑是否存在，``~/.vimrc`，当vim在启动时，如果没有找到vimrc或gvimrc，它缺省工作VI兼容的模式

### 安装vim主题
1. backup your old .vimrc if it is necessary
```
cp ~/.vimrc ~/.vimrc_bak
```
2. just get the file
```
curl https://raw.githubusercontent.com/wklken/vim-for-server/master/vimrc > ~/.vimrc
```
