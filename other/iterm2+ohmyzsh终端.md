---
title: "Iterm2+ohmyzsh终端"
date: 2020-11-09T09:40:52+08:00
draft: false  
categories: [other]
tags: [other]

---

# 配置mac终端

<!--more-->

### 安装iterm2 

[https://www.iterm2.com/index.html](https://www.iterm2.com/index.html) 下载iterm2 直接安装即可

### 安装ohmyzsh 

[https://ohmyz.sh/](https://ohmyz.sh/)  官网

使用命令安装 

```bash
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### 安装安装 powerlevel10k

*安装字体

```bash
brew tap homebrew/cask-fonts
brew cask install font-hack-nerd-font
```

设置iterm2 的字体

![](https://xing-blog.oss-accelerate.aliyuncs.com/blog/2020-11-11-152947.png)



#### powerlevel10k

### 设置.zshrc 

```bash
cat .zshrc(文件) 
if [[ -r "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh" ]]; then
  source "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh"
fi

ZSH_THEME="powerlevel10k/powerlevel10k"

plugins=(git  zsh-syntax-highlighting zsh-autosuggestions )

#防止每次输入 "colorls" 所以为 colorls 的常用操作设置别名
alias ll='colorls -lA --sd --gs --group-directories-first'
alias ls='colorls --group-directories-first'

# 一个shell 命令打印 tree 结构的方法查看当前目录下的目录树结构

alias tree="find . -print | sed -e 's;[^/]*/;|____;g;s;____|; |;g'"

# 缩短目录层级

POWERLEVEL9K_SHORTEN_DIR_LENGTH=
```

#### *下载powerlevel10k

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/themes/powerlevel10k
```

*****设置iterm2的字体，在运行p10k configure 时会自动安装一些字体**



![](https://xing-blog.oss-accelerate.aliyuncs.com/blog/2020-11-11-153047.png)



### 安装插件

#### *zsh-syntax-highlighting

```bash
brew install zsh-syntax-highlighting
```

#### *zsh-autosuggestions 

```bash
sudo git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

运行p10k configure 选择所对应的主题即可

***选择Y**

![](https://xing-blog.oss-accelerate.aliyuncs.com/blog/2020-11-11-153139.png)

![image-20201109211025357](https://xing-blog.oss-accelerate.aliyuncs.com/blog/2020-11-11-153318.png)



***选择2**

![image-20201109211050376](https://xing-blog.oss-accelerate.aliyuncs.com/blog/2020-11-11-153301.png)

**选择1** 

![image-20201109211122387](https://xing-blog.oss-accelerate.aliyuncs.com/blog/2020-11-11-153338.png)



**选择1** 

![image-20201109211232177](https://xing-blog.oss-accelerate.aliyuncs.com/blog/2020-11-11-153356.png)





**选择5**

![image-20201109211319518](https://xing-blog.oss-accelerate.aliyuncs.com/blog/2020-11-11-153410.png)



**选择2** 

![image-20201109211510321](https://xing-blog.oss-accelerate.aliyuncs.com/blog/2020-11-11-153431.png)



**选择1** 

![image-20201109211538985](https://xing-blog.oss-accelerate.aliyuncs.com/blog/2020-11-11-153733.png)



**选择Y**

![image-20201109211613712](https://xing-blog.oss-accelerate.aliyuncs.com/blog/2020-11-11-153904.png)



**最后选择1 和Y，会生成一个.p10k.zsh 的配置文件，成功之后就是我所选择的主题了**

![image-20201109211716953](https://xing-blog.oss-accelerate.aliyuncs.com/blog/2020-11-11-153916.png)

***可以根据需要随时调整主题**



