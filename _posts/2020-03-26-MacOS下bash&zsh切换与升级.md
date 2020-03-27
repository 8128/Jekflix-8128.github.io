---
layout: post
title:  MacOS下bash&zsh切换与升级
date:   2020-03-26 20:44:00
description: Catalina默认为zsh，你习惯了吗？
subtitle: 
comments: true
image: https://raw.githubusercontent.com/8128/PicGo/master/20200321004047.png
optimized_image: 
category: tutorial
tags:
  - tutorial
  - code
author: tty
paginate: false

---

zsh和bash都是shell，linux默认是bash，现在macOS最新版的系统Catalina默认是zsh。很多小伙伴的

## 切换系统默认shell

- 从bash切换到zsh

```bash
$ chsh -s /bin/zsh
```

输入密码成功切换，重启终端即可使用 zsh

- 从zsh切换到bash

```bash
$ chsh -s /bin/bash
```

重启终端即可。



bash 的环境变量是`.bash_profile`文件。
 zsh 的环境变量是`.zshrc`文件。

PS：如果从 bash 切换到 zsh，但想保留 bash 所设置的环境变量，可在 `.zshrc`文件末尾添加 `source ~/.bash_profile` 保存退出，并重启终端即可使用 bash 的环境变量

### 关闭bash已过时提示

**假如你还要继续后面的bash升级，你可以无视这一节，更新完bash提示自然会消失**

你在切换到bash之后，会不断收到terminal提示：

```shell
The default interactive shell is now zsh.

To update your account to use zsh, please run `chsh -s /bin/zsh`.
```

这就是苹果想让你去用默认的zsh。这倒也不奇怪，你放个旧版bash在这里，当然希望大家都去用新版zsh。假如你想关闭这条提示，你可以修改`.bash_profile`，如下

```bash
vim ~/.bash_profile
```

然后向其中添加：

```shell
# macOS Catalina

export BASH_SILENCE_DEPRECATION_WARNING=1
```

随后esc，输入wq，保存退出。重新打开就不会有警告了

## 升级MacOS bash

可能你是个复古派，或者你跟我一样懒得自己配环境（公司里的脚本都是按bash写的有没有！zsh改来改去很麻烦！）这时候你就需要转回bash

但是MacOS的bash版本不能更低了。MacOS上zsh是最新版，但bash却是

```bash
$ bash --version
GNU bash, version 3.2.57(1)-release (x86_64-apple-darwin19)
Copyright (C) 2007 Free Software Foundation, Inc.
```

我滴个龟龟，这版本是刚从土里掘出来的吧？可能苹果公司很有匠人精神，还在打磨这款经典bash。但是我不是匠人，我要升级。我们直接通过homebrew安装最新版bash

```bash
$ brew install bash
```

这时候我们查看一下安装成功了没有，然后发现系统里有俩bash

```bash
$ which -a bash

/usr/local/bin/bash
/bin/bash
```

这时候因为`/usr/local/bin/bash`是默认路径，假如你重启terminal你会发现你已经变成了新版bash。不过这还没完，unix系统会有安全权限等问题，我们需要把新安装的bash加入白名单，让它能成为login shell

```bash
$ sudo vim /etc/shells
```

之后再这个文件的底部加上`/usr/local/bin/bash`，结果如下

```
# List of acceptable shells for chpass(1).
# Ftpd will not allow users to connect who are not using
# one of these shells.

/bin/bash
/bin/csh
/bin/dash
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
/usr/local/bin/bash
```

最后我们将新shell设置为默认login shell

```bash
$ chsh -s /usr/local/bin/bash
```

以上命令只会更改你一个人的默认shell，加入你想更改所有人的

```bash
$ sudo chsh -s /usr/local/bin/bash
```

查看下自己现在的bash版本

```bash
$ echo $BASH_VERSION

5.0.16(1)-release
```

