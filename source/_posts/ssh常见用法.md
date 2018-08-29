---
title: ssh常见用法
date: 2018-08-29 12:58:07
tags:
    - linux

category: 
    - linux
---


ssh 公钥认证是ssh认证的方式之一,通过公钥认证可实现ssh免密码登陆,使用场景较多,如github代码提交校验，免密登录远程服务器等。

### 生成公钥和私钥
默认情况下，~/.ssh目录下会存放当前用户ssh配置认证相关的文件

使用 ssh-keygen命令生成ssh公钥认证所需的公钥和私钥文件
```shell
ssh-keygen

# 指定加密类型
ssh-keygen -t rsa

# 添加备注信息
ssh-keygen -t rsa  -C "comment"

# 指定文件名
ssh-keygen -t rsa  -C "comment" -f "test"

# 查看私钥对应的公钥信息
ssh-keygen -y -f private_key
```

### 免密登录秘钥配置
将公钥发送至服务器

```shell
# 将公钥加至服务器的authorized_keys文件中(确保该文件权限为600)
ssh-copy-id -i ~/.ssh/id_rsa.pub username@192.168.1.11

# 或者使用scp命令将文件传到服务器上,在服务器上操作
cat id_rsa.pub >> authorized_keys
```


### 远程登录服务器

```shell
# SSH 常用连接参数
#   -i 指定密钥路径  #默认在家路劲的.ssh/下
#   -p 指定SSH端口
#   -l 指定用户
#   -F 指定配置文件
#   -t 指定为终端迫使SSH客户端以交互模式工作，常配合expect使用

# 登录1.11服务器
ssh root@192.168.1.11

# 指定ssh端口
ssh -p 8888 root@192.168.1.11

# 登录远程服务器执行命令并退出(要执行的命令放在ssh命令后边)
ssh root@192.168.1.11 ls -hl

# 开启调试模式
ssh -v root@192.168.1.11

# 使用指定秘钥登录服务器
ssh -i ~/.ssh/id_rsa_11  root@192.168.1.11

# SSH构建跳板隧道(本机无登录0.10权限，但1.10有权限，通过1.10登录0.10服务器)
ssh –t 192.168.1.10 ssh 10.0.0.10

# 绑定ip 如果你的SSH客户端有多于两个以上的IP地址，你就不可能分得清楚到底是哪一个IP地址连接到了SSH服务。
ssh –b 192.168.1.10 root@10.0.0.10

# ssh远程端口映射，将外网端口映射到本地
ssh -R 8080:127.0.0.1:8080 username@12.34.56.78
```

### ssh 登录方式

#### 账号密码登录
```shell
+-----------+   1.send ssh request     +-----------+
|           |                          |           |
|           +------------------------> |           |
|           |                          |           |
|           |   2.send public key      |           |
|           |                          |           |
|  client   | <------------------------+  server   |
|           |                          |           |
|           |   3.encode send data     |           |
|           |                          |           |
|           +------------------------> |           |
|           |                          |           |
+-----------+                          +-----------+
```
1. 当客户端发起ssh请求，服务器会把自己的公钥发送给用户；
2. 用户会根据服务器发来的公钥对密码进行加密；
3. 加密后的信息回传给服务器，服务器用自己的私钥解密，如果密码正确，则用户登录成功

#### 秘钥登录
```shell
+------------------+                                 +------------------+
|                  |                                 |                  |
| 1.gen key pairs  |                                 |4.verify from auth|
|                  |                                 |orized_keys and   |
| 2.add server auth|   3.request with ip,name        |gen a random str  |
| orized_keys      | +-----------------------------> |                  |
|                  |                                 |                  |
|                  |                                 |                  |
|                  |                                 |                  |
|                  |                                 |                  |
|     client       |                                 |      server      |
|                  |                                 |                  |
|                  |                                 |                  |
| 6.use pubkey     |    5.send encode str            |                  |
| decode str       |  <----------------------------+ |                  |
|                  |                                 |                  |
|                  |                                 |8.use self key    |
|                  | 7.use server pubkey encode str  |decode str compare|
|                  | +-----------------------------> |with  origin str  |
|                  |                                 |                  |
+------------------+                                 +------------------+
```

1. 首先在客户端生成一对密钥（ssh-keygen）；
2. 并将客户端的公钥ssh-copy-id 拷贝到服务端；
3. 当客户端再次发送一个连接请求，包括ip、用户名；
4. 服务端得到客户端的请求后，会到authorized_keys中查找，如果有响应的IP和用户，就会随机生成一个字符串
5. 服务端将使用客户端拷贝过来的公钥进行加密，然后发送给客户端；
6. 得到服务端发来的消息后，客户端会使用私钥进行解密，然后将解密后的字符串发送给服务端；
7. 服务端接受到客户端发来的字符串后，跟之前的字符串进行对比，如果一致，就允许免密码登录。