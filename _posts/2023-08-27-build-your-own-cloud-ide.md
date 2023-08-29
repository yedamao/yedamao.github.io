---
layout: post
title: "build your own cloud IDE"
date: 2023-08-27 09:21:00 -0000
categories: vim tmux
---

# 定制自己专属的云端开发环境
---
# 目标🎯
- 使用VIM进行所有文本编辑(心中有剑)
- 开始拥有一套专属定制终端开发环境

---
# 挑选shell
⌨️

---
## oh-my-zsh
- 很多漂亮的主题
- 丰富的插件

---
### 插件
- autojump
- zsh-autosuggestions
- zsh-syntax-highlighting
- 各种命令工具的插件
---
## fish

---
# 挑选终端复用器 
守护天使👼

---
## 什么是终端复用器?
使用ssh登陆后, sshd会派生出一个shell进程, 用户通过和这个shell进程交互执行程序命令.

用户执行的程序命令一般都是当前shell进程的子进程, ssh连接断了之后,当时执行的程序都会退出.

---
## tmux
https://github.com/tmux/tmux/wiki/Getting-Started

### sessions
| 命令        | 功能                           |
| ----------- | ------------------------------ |
| tmux        | 启动tmux/创建默认session       |
| tmux attach | 进入之前的工作环境/默认session | 

---
### panes

> prefix key  默认 ctrl + b

| 命令                  | 功能               |
| --------------------- | ------------------ |
| prefix key + %        | 左右分屏           |
| prefix key + "        | 上下分屏           |
| prefix key + ⬆️⬇️⬅️➡️ | 分屏间移动         | 

---
### windows
| 命令                  | 功能               |
| --------------------- | ------------------ |
| prefix key + c        | 新建窗口           |
| prefix key + n        | 切换至上个窗口     |
| prefix key + p        | 切换至下个窗口     |

---
##  Zellij
https://zellij.dev

---
# 挑选编辑器 
![[Pasted image 20230815173539.png]]

---
## emacs
神的编辑器

伪装成编辑器的操作系统

---
## vim
编辑器之神

---
### long live interface
编辑器来来往往, vim接口永存

---
### 手中无剑心中有剑🗡️
- emacs evil mode
- IdeaVim
- VSCodeVim


---
搞什么 keones copilot,应该搞一个vim 比赛，传统组对vim 组起码提高30%编码效率

---
# 开始动手吧

---
## 准备工作🔨
一台服务器作为开发机

配置: 公司的开发机就ok (2C8G)

不动手应该就是看个热闹

---
## dotfiles
`sh -c "$(curl -fsLS git.io/chezmoi)" -- init --apply yedamao`

---
## 一键部署脚本
`sh -c "$(curl -fsSL https://raw.githubusercontent.com/yedamao/installer/main/installer.sh)"`

---
# vimtutor
---
## 梦开始的地方 ☁️
Vimtutor 十年前最超值的30min时间投资

---
Let's do this

---
# 好东西
## vim-slime

---
# reference
- [tmux使用教程](https://www.ruanyifeng.com/blog/2019/10/tmux.html)
- [vim-slime](https://github.com/jpalardy/vim-slime)
- <Unix环境高级编程> chapter 9.9 shell执行程序
