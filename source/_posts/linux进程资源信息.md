---
title: linux进程资源信息
date: 2017-04-17 01:24:36
tags:
---
## Linux进程与资源管理
Linux最棒的地方在于它的多用户多任务环境，记录下多用户多任务相关的几个常用命令。

### 前后台进程及切换
有时候我们并不一定要在屏幕前工作，这时就需要使用到背景工作管理的一些指令，如 `&`，`ctrl+z`。

如果想让程序在背景下执行，可以使用 `&`,由于是背景，该程序的输入并不会显示在屏幕上。

语法格式: `command &`

``` shell
# 程序进入背景执行
find / -name test &

# fg 将程序拉回屏幕前执行
fg
```

也可以通过 `ctrl+z` 将正在运行的工作丢到背景下，放在背景下的最大好处就是不会被 `ctrl+c` 指令中断。

如果出现某个时候需要暂时退出vim，但又不想保存退出，可 `ctrl+z` 暂时退出

``` shell
# 程序进入背景执行
vim a.js

# 将vim放到背景中，并没有退出
ctrl+z 
```

可使用 `jobs` 可以查看任务列表，配合 `bg` ，`fg`将程序拉回屏幕。

查看背景程序：`jobs`

``` shell
# 查看背景程序
jobs
[1]  + suspended  vim a.js

# 恢复vim编辑
fg %1
```

`fg & bg `恢复背景进程

``` shell
#number 为jobs中方括号中的编号
fg  %number

#将背景程序中的程序由stopped变成running
bg %number
```

`kill` 杀掉背景程序中的程序

```shell
kill -signal %number
signal
    -1 :重新读取参数配置（类似reload）
    -2 :类似Ctrl+c中断工作
    -9 :强制杀掉进程
    -15 :停止一个工作（默认值）
    
#杀掉vim 进程    
kill -9 %1    
```

### 查看进程
`ps` 查看进程信息

```shell
ps -aux
参数说明
    a :显示所有程序
    u :列出所有用户程序
    x :列出所有tty程序
    
# 显示进程信息    
ps -aux

USER               PID  %CPU %MEM      VSZ    RSS   TT  STAT STARTED      TIME COMMAND
nali              2557   3.0  1.2  2832304  49180   ??  S    五04下午   4:54.94 /Applications/iTerm 2.app/Contents/MacOS/iTerm2
nali               920   1.6  0.8  2939412  34404   ??  S    四07下午   5:26.69 /Applications/QQ.app/Contents/MacOS/QQ   

#干掉进程 
kill -9 pid

```
另外一个显示进程的命令是 `top` 可动态显示。
查看内存使用：`free`
查看系统资源：`sar`




### 程序优先级
由于CPU资源有限，优先级高的程序会先获取CPU资源。

```shell
ps -l

UID   PID  PPID        F CPU PRI NI       SZ    RSS WCHAN     S             ADDR TTY           TIME CMD
501  9005  2557     4006   0  31  0  2507116   9148 -      Ss                  0 ttys001    0:00.07 /Applications/iTerm 2.app/Contents/MacOS/iTerm2 --server login 

#PRI 代表程序优先级，越小越先被执行
NI 代表nice的值
```
*PRI越小越优先被执行*

`PRI(new) = PRI(old) + nice`

>一般用户可用的nice值 0~19
root可用的nice值-20~19

调整程序的的优先级
`nice [-n number] command`

调整运行中程序的优先级
`renice number PID`

### 查看系统相关信息
`uname` 查看系统信息

```shell 
uname 
参数信息：
    -a :列出所有信息
    -p :列出CPU信息
    -r :列出核心版本信息
    -n :列出版本信息 
```

`dmesg` 查看启动一闪而过的信息

`uptime` 显示开机时间及负载相关信息

`last` 显示登录信息

`hostname` 显示主机名


`who` 查看当前系统上的用户(只列出用户名及登录时间)

`w` 查看当前系统上的用户(列出用户名及登录时间 + 源地址IP + 工作项 + ..)

`whoami` 显示当前登录用户名


`ntpdate` 网络校时

```shell
# 校时服务器 time.stdtime.gov.tw

# 执行校时命令
ntpdate time.stdtime.gov.tw

# 可配合crontab定时校时
```







