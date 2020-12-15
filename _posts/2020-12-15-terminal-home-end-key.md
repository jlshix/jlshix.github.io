---
title: "终端 home end 键变为 ~ 的问题"
subtitle: ""
layout: post
author: "jlshix"
header-style: text
tags:
  - bash
  - terminal
---

近期由于 VPN 的平台限制, 不得不在 windows 上使用 xshell 连接一台 Linux 服务器.
而我使用的是 Mac.

一顿操作之后终于连接好了正常使用. 但在使用过程中, 发现在终端输入命令想使用 home 或 end 跳到
行首或行尾时, 却打印出一个 `~`.

一度以为是系统匹配问题, 直到进入某 docker 容器后习惯性按 home 或 end 却正常工作, 确认是
服务器本身的配置问题. 一番搜索后发现是 `inputrc` 文件的配置问题.

此文件作为 Readline 库的配置文件读入, 此文件作为 Bash 等大多数 shell 的输入相关的库.

默认的配置文件在 `/etc/inputrc`, 若要单独定制可复制到 `~/.inputrc` 进行修改.
对于 home 和 end 只需要增加以下两行即可:

```shell
"\e[1~": beginning-of-line
"\e[4~": end-of-line
```

参见:

1. https://www.linuxquestions.org/questions/slackware-14/home-and-end-keys-in-a-terminal-803635/

2. https://wiki.archlinux.org/title/Home_and_End_keys_not_working

3. https://www.gnu.org/software/bash/manual/html_node/Readline-Init-File-Syntax.html