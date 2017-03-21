---
title: tmux
date: 2017-03-21 12:34:33
tags:
 - 工具
---

上篇讲到了 [iTerm2的基本使用](http://www.jianshu.com/p/3436bcb17a03) ，iTerm2能够让我们迅速从不同的窗口中切换。通过 iTerm2 ，我们可以在不同窗口中愉快用 ssh 登录到远处服务器去操作，但由于临时断网或其他原因，远程连接就会中断，如想继续之前的工作，又得重头来过T_T ,但是iTerm2有个 tmux 的配合，再也不用担心异常中断了，它可以让我们迅速恢复到之前的现场环境。

![tmux 窗格](http://upload-images.jianshu.io/upload_images/3778142-7340859f88245de4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[tmux](http://tmux.github.io/) 是一个终端复用软件，它也有类似iTerm2的分屏功能。不仅如此，我们还可以通过tmux随时断开会话或者接入会话，接下一起来看下tmux 一些基本使用方法。

### 入门须知
- server服务器：输入tmux命令时就开启了一个服务器，可以开启多个会话。
- session: 管理多个window的会话
- window: 一个window就是整个屏幕
- pane: 一个window可以被横向或纵向分割为多个窗格

即一个session可包含多个窗口，一个窗口中可包含多个窗格。
一般情况下 tmux 中所有的快捷键都需要和前缀快捷键 ```ctrl + b``` 来组合使用。


### 安装tmux
``` shell
brew install tmux
```

 ### 运行 tmux
开启了一个 tmux 的会话，默认会新建一个窗口和一个窗格，窗口左下角会有一些标识信息。
``` shell
tmux
```

![tmux默认窗口](http://upload-images.jianshu.io/upload_images/3778142-64ef52c19d9ef73e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 会话常用命令
``` shell
#在正常终端模式下使用 tmux 建立会话并命名
tmux new -s abc

#休眠会话 返回主shell- 在正常终端模式下，使某个编号的会话强制休眠，编号用的是 tmux ls 命令时所列出的每一行的最前面的那个编号
tmux detach -t 编号
tmux detach -s 名称

#恢复会话
tmux attach -t 编号
## 也可简写成
tmux a -t 编号
tmux attach -s test

#重命名会话名称
tmux rename -t test dev

#关闭会话
tmux kill-session -t abc

# 完全退出，关闭所有的会话
tmux kill-server
```


### 窗口常用命令
假设当前默认前缀为 : Ctrl+b
 ``` shell
{前缀} c 创建新窗口

{前缀} n 选择下一个窗口

{前缀} p 选择前一个窗口

{前缀} l 最近一次活跃窗口之间进行切换

{前缀} 0~9 选择几号窗口

{前缀} , 重命名窗口

{前缀} . 更改窗口的编号，但只能更改成未使用的编号，所以要交换窗口的话，得更改多次进行交换

{前缀} & 关闭窗口

{前缀} w 以菜单方式显示及选择窗口

{前缀} f 在所有窗口中查找内容
```


### 窗格常用命令
``` shell
{前缀} " 模向分隔面板

{前缀} % 纵向分隔面板

{前缀} o 跳到下一个分隔面板

{前缀} x 关闭面板

{前缀} ; 切换到最后一个使用的面板

{前缀} 上下键 上一个及下一个分隔面板

{前缀} 空格键 切换面板布局

```

### 其他
``` shell
{前缀} t 显示时钟

{前缀} m 鼠标切换窗格
```
看到这里，尝试后是不是觉得很奇怪，自己打开的窗口，怎么和本页面第一张图不一样！不一样！ 首先恭喜你已经掌握了tmux基本操作，接下来咱们看下tmux高级用法。



###高级配置
tmux 自定义配置[参见 .tmux](https://github.com/gpakosz/.tmux)

---
tmux窗口左下角状态显示用到了Powerline及Powerline字体，接着咱们来美化下tmux窗口。

1.安装 Python
```shell
brew install python
```
2.下载 [powerline](https://github.com/powerline/powerline)
```shell
sudo pip install powerline-status
```

3.配置 Powerline 到终端
```shell
#查看安装路径
pip show powerline-status

#配置 .bash_profile 文件,添加以下行
. /Powerline安装路径/powerline/bindings/bash/powerline.sh
source .bash_profile
```

4.安装专用于 [Powerline](https://github.com/powerline/fonts) 的字体
然后在 iTerm 2的偏好设置里的Profile选项卡里把字体设置为以 Powerline 结尾的字体就大功告成了。


PS：如果满足不了你的操作体验，你还可以：
安装配色方案 [solarized](https://github.com/altercation/solarized)
安装zsh主题 [agnoster-fcamblor](https://github.com/fcamblor/oh-my-zsh-agnoster-fcamblor)