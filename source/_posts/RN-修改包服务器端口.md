---
title: '[RN]修改包服务器端口'
date: 2017-09-11 00:38:23
tags:
    - js
    - react native

category: 
    - js
    - react native
---

开发React Native时，启动包服务器的默认端口是`8081`，如果跟现有服务器端口冲突的话，可以修改成想要的端口，修改内容如下：

### 修改包服务器端口号
包服务器启动监听端口，在项目的`node_modules/react-native/local-cli/server/server.js中`

``` js
options: [{
    command: '--port [number]',
    default: 8081,
    parse: (val) => Number(val),
  },

  ...
  ]
```

由此可见我可以在运行时`react-native start --port=8082`动态指定端口

### 修改应用程序启动时，读取逻辑代码的的地址
在`ios/Demo01/AppDelegate.m`中找到下面代码

```
// 修改后代码
  jsCodeLocation = [NSURL URLWithString:@"http://localhost:8081/index.ios.bundle"];
// jsCodeLocation = [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index.ios" fallbackResource:nil];
```


### 修改websocket监听端口
在项目的`node_modules/react-native/Libraries/WebSocket/RCTWebSocketExecutor`中修改下端口。

```
- (void)setUp
{
  if (!_url) {
    NSInteger port = [[[_bridge bundleURL] port] integerValue] ?: 8081;
    NSString *host = [[_bridge bundleURL] host] ?: @"localhost";
    NSString *URLString = [NSString stringWithFormat:@"http://%@:%zd/debugger-proxy?role=client", host, port];
    _url = [RCTConvert NSURL:URLString];
  }
```
