---
title: sed常用操作命令
date: 2017-04-26 20:46:08
tags:
    - linux

category: 
    - linux
---

#### SED命令

SED的英文全称是 Stream EDitor，它是一个简单而强大的文本解析转换工具。

#### 原理
1. 读一行到缓存区处理（模式空间）
2. 执行sed命令
3. 输出结果接着继续处理下一行（重复此流程直至结束）


#### 用途
* 文本替换
* 选择性的输出文本文件
* 从文本文件的某处开始编辑
* 无交互式的对文本文件进行编辑等

#### 命令格式
```shell
sed [option]  command files
option:
    -e:指定要执行的命令，使用该参数，我们可以指定多个命令
    -n:默认情况下，模式空间中的内容在处理完成后将会打印到标准输出，该选项用于阻止该行为
    -f:指定包含要执行的命令的脚本文件(一行一个命令,类似多个-e)
    
command：行定位（或正则）+ sed命令 （命令使用单引号包裹）

sed -e '1d' -e '2d' -e '5d' demo.txt
```
在SED中使用的命令会作用于文本数据的所有行。如果只想将命令作用于特定的行或者某些行，则需要使用 "行寻址" 功能。

##### 行寻址两种形式：

```
1.以数字形式表示的行区间
sed -n '3p' demo.txt 

# 区间 
sed -n '2,5 p' demo.txt 

# $ 表示自后一行 ^ 标识开始
sed -n '3,$ p' demo.txt

2.以文本模式来过滤行
sed -n '/Alchemist/, 5 p' books.txt
```


##### 操作符 + , ~
加号（+）操作符，它可以与逗号（,）操作符一起使用

```
# 从第二行开始下4行
sed -n '2,+4 p' demo.txt  
```
使用波浪线操作符（~）指定地址范围，它使用M~N（步长）的形式，告诉SED应该处理M行开始的每N行。

```
# 50~5匹配行号5，10，15等
sed -n '5~5 p' demo.txt > a.txt 
```

>sed操作不影响源文件，可用重定向覆写源文件.


#### sed操作命令
##### 打印命令 `p` 
打印命令p通常和-n选项配合

```
# 打印1~10行
sed -n '1,10p' demo.txt

# 打印非10行 
sed -n '10!p' demo.txt
```


##### 插入命令 `a/i` 

```
# 第五行后插入hello
sed -n '5a hello' demo.txt

# 第五行前插入hello
sed -n '5i hello' demo.txt
```
##### 替代 `c` 

```
# 第四行替换成hi
sed -n '4c hi' demo.txt
```

##### 删除 `d` 

```
# 删除第十行
sed -n '10d' demo.txt 

# 删除空行
sed '^&d' demo.txt  
```


##### 替换 `s` 

```
# false 替换成true
sed 's/false/true/g' demo.txt
```


##### 执行多条命令 `{ ；}` 

```
# 多条命令使用{}，命令间用分号分隔
sed -n '{p;n;p}' demo.txt
```

##### 读取下一行到模式空间里去 `n` 

```
# false 替换成true
sed 's/false/true/g' demo.txt
```

>h: 将当前模式空间的内容保存到保存空间
H：将当前模式空间中的内容追加到 保持空间 
g : 将保持空间复制到模式空间
G: 将保持空间附加到模式空间
x：用于交换模式空间和保持空间中的内容


##### 跳转 `b` 

```
# 类似goto跳转
sed -n 'h;n;H;x;s/\n/, /;/Paulo/!b Print; s/^/- /; :Print;p' demo.txt 
```


##### 创建分支 `t` 

```
# 创建分支;只有当前置条件成功的时候，t 命令才会跳转到该标签
sed -n 'h;n;H;x; s/\n/, /; :Loop;/Paulo/s/^/-/; /----/!t Loop; p' demo.txt 
```


##### 替换字符 `&` 

```
# & 为匹配的字符
sed 's/w/&123' demo.txt
```


##### 大小写转化 `\u \U \l \L` 

```
# & 为匹配的字符
sed 's/w/\u&123/' demo.txt
```

##### 字符一对一转化 `y` 

```
# [address]y/inchars/outchars/ 
# inchars中的第一个字符会被转换为outchars中的第一个字符，第二个字符会被转换成outchars中的第二个字符
$ echo "1 5 15 20" | sed 'y/151520/IVXVXX/'
```

##### 显示隐藏字符命令 `l` 

```
# [address1[,address2]]l [len] 
# 可显示 空格 tab，实现文本按照指定的宽度换行
sed -n 'l 25' books.txt
```

##### 真正匹配引用 `\x` 

```
# () () () ...  \1 \2 \3 ...用\x表示前面的括号内容
# 可显示 空格 tab，实现文本按照指定的宽度换行
sed -r 's/(^[a-z-_]+):x:([0-9]+):([0-9]+).*$/user:\1  uid:\2  gid:\3/' /etc/passwd罗列出用户名 uid gid
```

##### 复制指定文件内容到匹配行 `r` 

```
# 读取文件内容显示到指定行
sed -n '1r a.txt' b.txt
```

##### 将匹配行写入到文件中 `w` 

```
# 将demo.txt第一行写入到a.txt中
sed -n '1w a.txt' demo.txt
```
##### 执行外部命令  `e` 

```
# e 执行外部命令 比如 date
sed '3 e date' books.txt
```


##### 排除命令  `!` 

```
# 匹配Paulo的行不打印
sed -n '/Paulo/!p' books.txt
```

##### 输出行号  `=` 

```
# [address1[,address2]]=
sed '=' demo.txt
```

##### 退出sed  `q` 

```
# 执行完第10行退出
sed -n '10q' 

# q命令也支持提供一个value(作为程序的返回代码返回)

sed '/The Alchemist/ q 100' books.txt
echo $? 
```

##### 多行命令  `P,D,N` 

```
# N：将数据流中的下一行加进来创建一个多行组来处理,N并不会清除、输出模式空间的内容，而是采用了追加模式
sed -n '10q' 

# D：删除多行组中的一行,只删除模式空间中的第一行。该命令会删除到换行符（含 换行符）为止的所有字符

# P：打印多行组中的一行 
```




[参考：三十分钟学会SED](https://github.com/mylxsw/growing-up/blob/master/doc/%E4%B8%89%E5%8D%81%E5%88%86%E9%92%9F%E5%AD%A6%E4%BC%9ASED.md)

