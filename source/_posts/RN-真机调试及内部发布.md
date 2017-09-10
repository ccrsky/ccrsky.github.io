---
title: '[RN]真机调试及内部发布'
date: 2017-09-11 01:15:47
tags:
    - js
    - react native

category: 
    - js
    - react native
---

一般我们开发RN都是在模拟器上调试的，那么真机是如何调试的呢？

### 真机调试

1. 配置好签名证书，并将包环境设置成Debug模式
2. 将`AppDelegate.m`中的localhost修改成对应的IP地址，将调试真机插在电脑上，Xcode模拟器选择真机。
3. 运行编译命令，真机摇一摇手机即可调出调试面板

### 内部发布
内部发布是上App Store之前的一步，开发完RN项目后可供内部人员安装和使用

1. 修改`AppDelegate.m`中jsCodeLocation为 `jsCodeLocation = [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index.ios" fallbackResource:nil];`
2. 在项目下运行 `curl http://localhost:8081/index.ios.bundle -o main.jsbundle`
或者运行`react-native bundle`

此时就是在项目目录下生成`main.jsbundle`文件，这个文件将所有js文件都打包在一起了，然后配置好证书，再编译得到ipa离线包，安装后即可在真机跑起来了。