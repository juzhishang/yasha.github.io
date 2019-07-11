---
title: brew install 卡住的问题
date: 2019-07-09 23:48:49
tags: 小问题
---

在使用brew安装node时，发现安装一直卡在brew升级这一步
```
➜  ~ brew install node
Updating Homebrew...
```

##### 最简单的解决办法是`ctrl + C`,这样它会取消更新，然后继续执行安装node的步骤。

但接下来的这步也可能报错。（然后就看它报什么错呗）

##### 第二个方案：尝试换源

brew跟以下3个仓库有关：

+ brew.git

+ homebrew-core.git

+ homebrew-bottles

可以将它们换成阿里的源。

1. brew.git

```
# 替换
cd "$(brew --repo)"
git remote set-url origin https://mirrors.aliyun.com/homebrew/brew.git


# 还原
cd "$(brew --repo)"
git remote set-url origin https://github.com/Homebrew/brew.git
```

2、homebrew-core.git
```
# 替换
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.aliyun.com/homebrew/homebrew-core.git

# 还原
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://github.com/Homebrew/homebrew-core.git
```

3、homebrew-bottles

首先要看使用的shell类型。
```
echo $SHELL
// /bin/zsh
```
看到我目前使用的是zsh。

如果是zsh,替换/还原的命令是：
```
# 替换
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.aliyun.com/homebrew/homebrew-bottles' >> ~/.zshrc
source ~/.zshrc

# 还原
vi ~/.zshrc
# 然后，删除HOMEBREW_BOTTLE_DOMAIN这一配置
source ~/.zshrc
```
如果是bash，替换/还原的命令是：
```
# 替换 homebrew-bottles 访问 URL:
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.aliyun.com/homebrew/homebrew-bottles' >> ~/.bash_profile
source ~/.bash_profile


# 还原
vi ~/.bash_profile
# 然后，删除HOMEBREW_BOTTLE_DOMAIN这一配置
source ~/.bash_profile
```


##### 题外话：brew的重装(没事别这么干，因为有点耗时，卸载后之前通过brew安装的应用也会被删除)

>1 卸载brew
>执行命令：`/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall)"`

>2 安装brew
>执行命令：`/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" `
