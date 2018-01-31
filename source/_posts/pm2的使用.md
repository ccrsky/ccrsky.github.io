---
title: pm2的使用
date: 2018-01-31 20:52:26

tags:
    - node
    - js

category: 
    - js
---

pm2是一个高级node进程管理工具，通过它不仅可以管理node应用，也可以管理Python等其他语言的应用；本文是看完官网后做的笔录，适合快速入门pm2命令行工具的使用，具体细节可参考[官网介绍](http://pm2.keymetrics.io/docs/usage/quick-start/)

### pm2使用

#### 快速使用
安装

```
npm install pm2@latest -g
```

使用

```
pm2 start app.js
```

通过配置文件管理多个应用

```
# 创建 process.yml内容如下：
apps:
  - script   : app.js
    instances: 4
    exec_mode: cluster
  - script : worker.js
    watch  : true
    env    :
      NODE_ENV: development
    env_production:
      NODE_ENV: production
      
      
# 运行应用
pm2 start process.yml

# 随系统启动
pm2 startup
```
与`pm2`相关的所有配置位于`$HOME/.pm2`下

* logs 应用log
* pids 应用pid
* sock 套接字文件
* conf.js 配置文件


更新

```
# 全局安装
npm install pm2@latest -g

# 内存更新
pm2 update
```



命令清单

```
# Fork mode
pm2 start app.js --name my-api # Name process

# Cluster mode
pm2 start app.js -i 0        # Will start maximum processes with LB depending on available CPUs
pm2 start app.js -i max      # Same as above, but deprecated.

# Listing

pm2 list
# Or
pm2 [list|ls|l|status]

# 显示进程详情
pm2 show 0

pm2 list               # Display all processes status
pm2 jlist              # Print process list in raw JSON
pm2 prettylist         # Print process list in beautified JSON

pm2 describe 0         # Display all informations about a specific process

pm2 monit              # Monitor all processes

# Logs

pm2 logs [--raw]       # Display all processes logs in streaming
pm2 flush              # Empty all log files
pm2 reloadLogs         # Reload all logs

# Actions

pm2 stop all           # Stop all processes
pm2 restart all        # Restart all processes

pm2 reload all         # Will 0s downtime reload (for NETWORKED apps)

pm2 stop 0             # Stop specific process id
pm2 restart 0          # Restart specific process id

# 重启服务
pm2 restart myapp          # Restart specific process id

# 根据应用名称删除
pm2 delete myapp          
pm2 delete 0           # Will remove process from pm2 list
pm2 delete all         # Will remove all processes from pm2 list

# Misc

pm2 reset <process>    # Reset meta data (restarted time...)
pm2 updatePM2          # Update in memory pm2
pm2 ping               # Ensure pm2 daemon has been launched
pm2 sendSignal SIGUSR2 my-app # Send system signal to script
pm2 start app.js --no-daemon
pm2 start app.js --no-vizion
pm2 start app.js --no-autorestart

# 给应用传递参数
pm2 start app.js -- -a 23  # Pass arguments '-a 23' argument to app.js script
pm2 start app.js --node-args="--debug=7001" # --node-args to pass options to node V8

pm2 start app.js --log-date-format "YYYY-MM-DD HH:mm Z"    # Log will be prefixed with custom time format

# 以json文件方式启动
pm2 start app.json                # Start processes with options declared in app.json

# 指定多应用中的具体应用
pm2 reload process.json --only api  
       
# 自定义日志文件                                    
pm2 start app.js -e err.log -o out.log  # Start and specify error and out log

# PM2 2.4.0后 可以通过正则批量操作
pm2 restart/delete/stop/reload  /http-[1,2]/

```

启动其他语言应用

```
# 启动其他语言的进程
pm2 start echo.php
pm2 start echo.py
pm2 start echo.sh

# 映射关系 - 解释器
{
  ".sh": "bash",
  ".py": "python",
  ".rb": "ruby",
  ".coffee" : "coffee",
  ".php": "php",
  ".pl" : "perl",
  ".js" : "node"
}
# 通过配置文件的方式启动，非js应用必须指定exec_mode: 'fork_mode' and exec_interpreter至对应的解释器
{
  "apps" : [{
    "name"       : "bash-worker",
    "script"     : "./a-bash-script",
    "exec_interpreter": "bash",   
    "exec_mode"  : "fork_mode"
  }, {
    "name"       : "ruby-worker",
    "script"     : "./some-ruby-script",
    "exec_interpreter": "ruby",
    "exec_mode"  : "fork_mode"
  }]
}

```

控制应用内存大小

```
# 启动应用 显示内存大小
pm2 start big-array.js --max-memory-restart 20M

# JSON
{
  "name"   : "max_mem",
  "script" : "big-array.js",
  "max_memory_restart" : "20M"
}

# 使用脚本方式控制内存大小
pm2.start({
  name               : "max_mem",
  script             : "big-array.js",
  max_memory_restart : "20M"
}, function(err, proc) {
  // Processing
});

# 单位  K(ilobyte), M(egabyte), G(igabyte)
50M
50K
1G

```

### 集群模式
根据cpus个数生成多个应用实例，提高应用的性能及可靠性

```
pm2 start app.js -i max
# max:pm2自动检测cpus个数

# 通过配置 js/yaml/json file
{
  "apps" : [{
    "script"    : "api.js",
    "instances" : "max",
    "exec_mode" : "cluster"   
  }]
}
pm2默认不会负载均衡，需要制定 exec_mode为 cluster

-i 参数项
0/max: = CPUs
-1 : CPUs - 1
number: <= CPUS

```

应用平滑关闭

```
process.on('SIGINT', function() {
   db.stop(function(err) {
     process.exit(err ? 1 : 0);
   });
});
```

### 启动配置文件

#### 配置文件格式

* javascript
* json
* yaml

生成配置文件

```
# 生成 ecosystem.config.js
pm2 ecosystem

# 生成文件不带注释
pm2 ecosystem simple

```

JavaScript格式

```
module.exports = {
  apps : [{
    name        : "worker",
    script      : "./worker.js",
    watch       : true,
    env: {
      "NODE_ENV": "development",
    },
    env_production : {
       "NODE_ENV": "production"
    }
  },{
    name       : "api-app",
    script     : "./api.js",
    instances  : 4,
    exec_mode  : "cluster"
  }]
}
```

json格式

```
{
  "apps" : [{
    "name"        : "worker",
    "script"      : "./worker.js",
    "watch"       : true,
    "env": {
      "NODE_ENV": "development"
    },
    "env_production" : {
       "NODE_ENV": "production"
    }
  },{
    "name"       : "api-app",
    "script"     : "./api.js",
    "instances"  : 4,
    "exec_mode"  : "cluster"
  }]
}
```
yaml格式

```
apps:
  - script   : ./api.js
    name     : 'api-app'
    instances: 4
    exec_mode: cluster
  - script : ./worker.js
    name   : 'worker'
    watch  : true
    env    :
      NODE_ENV: development
    env_production:
      NODE_ENV: production
```
根据配置文件管理应用

```
pm2 start ecosystem.config.js

# 启动配置文件中具体某个应用
pm2 start ecosystem.config.js --only worker-app

# 停止配置文件中指定的所有应用
pm2 stop ecosystem.config.js

# 重启所有应用
pm2 start   ecosystem.config.js
## Or
pm2 restart ecosystem.config.js

# 重新加载所有应用
pm2 reload ecosystem.config.js

# 删除所有应用
pm2 delete ecosystem.config.js

```
更新环境变量信息

```

pm2 restart ecosystem.config.js --update-env
pm2 startOrReload ecosystem.config.js --update-env

# 根据参数自动注入环境变量  env_<name>
pm2 start process.json --env production
```

#### 配置参数列表
{% raw %}
<table>
  <thead>
    <tr>
      <th style="text-align: left">Field</th>
      <th style="text-align: center">Type</th>
      <th style="text-align: center">Example</th>
      <th style="text-align: left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: left">name</td>
      <td style="text-align: center">(string)</td>
      <td style="text-align: center">“my-api”</td>
      <td style="text-align: left">application name (default to script filename without extension)</td>
    </tr>
    <tr>
      <td style="text-align: left">script</td>
      <td style="text-align: center">(string)</td>
      <td style="text-align: center">”./api/app.js”</td>
      <td style="text-align: left">script path relative to pm2 start</td>
    </tr>
    <tr>
      <td style="text-align: left">cwd</td>
      <td style="text-align: center">(string)</td>
      <td style="text-align: center">“/var/www/”</td>
      <td style="text-align: left">the directory from which your app will be launched</td>
    </tr>
    <tr>
      <td style="text-align: left">args</td>
      <td style="text-align: center">(string)</td>
      <td style="text-align: center">“-a 13 -b 12”</td>
      <td style="text-align: left">string containing all arguments passed via CLI to script</td>
    </tr>
    <tr>
      <td style="text-align: left">interpreter</td>
      <td style="text-align: center">(string)</td>
      <td style="text-align: center">“/usr/bin/python”</td>
      <td style="text-align: left">interpreter absolute path (default to node)</td>
    </tr>
    <tr>
      <td style="text-align: left">interpreter_args</td>
      <td style="text-align: center">(string)</td>
      <td style="text-align: center">”–harmony”</td>
      <td style="text-align: left">option to pass to the interpreter</td>
    </tr>
    <tr>
      <td style="text-align: left">node_args</td>
      <td style="text-align: center">(string)</td>
      <td style="text-align: center">&nbsp;</td>
      <td style="text-align: left">alias to interpreter_args</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td style="text-align: left">instances</td>
      <td style="text-align: center">number</td>
      <td style="text-align: center">-1</td>
      <td style="text-align: left">number of app instance to be launched</td>
    </tr>
    <tr>
      <td style="text-align: left">exec_mode</td>
      <td style="text-align: center">string</td>
      <td style="text-align: center">“cluster”</td>
      <td style="text-align: left">mode to start your app, can be “cluster” or “fork”, default fork</td>
    </tr>
    <tr>
      <td style="text-align: left">watch</td>
      <td style="text-align: center">boolean or []</td>
      <td style="text-align: center">true</td>
      <td style="text-align: left">enable watch &amp; restart feature, if a file change in the folder or subfolder, your app will get reloaded</td>
    </tr>
    <tr>
      <td style="text-align: left">ignore_watch</td>
      <td style="text-align: center">list</td>
      <td style="text-align: center">[”[\/\\]\./”, “node_modules”]</td>
      <td style="text-align: left">list of regex to ignore some file or folder names by the watch feature</td>
    </tr>
    <tr>
      <td style="text-align: left">max_memory_restart</td>
      <td style="text-align: center">string</td>
      <td style="text-align: center">“150M”</td>
      <td style="text-align: left">your app will be restarted if it exceeds the amount of memory specified. human-friendly format : it can be “10M”, “100K”, “2G” and so on…</td>
    </tr>
    <tr>
      <td style="text-align: left">env</td>
      <td style="text-align: center">object</td>
      <td style="text-align: center">{“NODE_ENV”: “development”, “ID”: “42”}</td>
      <td style="text-align: left">env variables which will appear in your app</td>
    </tr>
    <tr>
      <td style="text-align: left">env_<env_name></env_name></td>
      <td style="text-align: center">object</td>
      <td style="text-align: center">{“NODE_ENV”: “production”, “ID”: “89”}</td>
      <td style="text-align: left">inject <env_name> when doing pm2 restart app.yml --env <env_name></env_name></env_name></td>
    </tr>
    <tr>
      <td style="text-align: left">source_map_support</td>
      <td style="text-align: center">boolean</td>
      <td style="text-align: center">true</td>
      <td style="text-align: left">default to true, [enable/disable source map file]</td>
    </tr>
    <tr>
      <td style="text-align: left">instance_var</td>
      <td style="text-align: center">string</td>
      <td style="text-align: center">“NODE_APP_INSTANCE”</td>
      <td style="text-align: left"><a href="http://pm2.keymetrics.io/docs/usage/environment/#specific-environment-variables">see documentation</a></td>
    </tr>
  </tbody>
  
  <tbody>
    <tr>
      <td style="text-align: left">log_date_format</td>
      <td style="text-align: center">(string)</td>
      <td style="text-align: center">“YYYY-MM-DD HH:mm Z”</td>
      <td style="text-align: left">log date format (see log section)</td>
    </tr>
    <tr>
      <td style="text-align: left">error_file</td>
      <td style="text-align: center">(string)</td>
      <td style="text-align: center">&nbsp;</td>
      <td style="text-align: left">error file path </td>
    </tr>
    <tr>
      <td style="text-align: left">out_file</td>
      <td style="text-align: center">(string)</td>
      <td style="text-align: center">&nbsp;</td>
      <td style="text-align: left">output file path </td>
    </tr>
    <tr>
      <td style="text-align: left">combine_logs</td>
      <td style="text-align: center">boolean</td>
      <td style="text-align: center">true</td>
      <td style="text-align: left">if set to true, avoid to suffix logs file with the process id</td>
    </tr>
    <tr>
      <td style="text-align: left">merge_logs</td>
      <td style="text-align: center">boolean</td>
      <td style="text-align: center">true</td>
      <td style="text-align: left">alias to combine_logs</td>
    </tr>
    <tr>
      <td style="text-align: left">pid_file</td>
      <td style="text-align: center">(string)</td>
      <td style="text-align: center">&nbsp;</td>
      <td style="text-align: left">pid file path (default to $HOME/.pm2/pid/app-pm_id.pid)</td>
    </tr>
  </tbody>
  
  <tbody>
    <tr>
      <td style="text-align: left">min_uptime</td>
      <td style="text-align: center">(string)</td>
      <td style="text-align: center">&nbsp;</td>
      <td style="text-align: left">min uptime of the app to be considered started</td>
    </tr>
    <tr>
      <td style="text-align: left">listen_timeout</td>
      <td style="text-align: center">number</td>
      <td style="text-align: center">8000</td>
      <td style="text-align: left">time in ms before forcing a reload if app not listening</td>
    </tr>
    <tr>
      <td style="text-align: left">kill_timeout</td>
      <td style="text-align: center">number</td>
      <td style="text-align: center">1600</td>
      <td style="text-align: left">time in milliseconds before sending <a href="http://pm2.keymetrics.io/docs/usage/signals-clean-restart/#cleaning-states-and-jobs">a final SIGKILL</a></td>
    </tr>
    <tr>
      <td style="text-align: left">wait_ready</td>
      <td style="text-align: center">boolean</td>
      <td style="text-align: center">false</td>
      <td style="text-align: left">Instead of reload waiting for listen event, wait for process.send(‘ready’)</td>
    </tr>
    <tr>
      <td style="text-align: left">max_restarts</td>
      <td style="text-align: center">number</td>
      <td style="text-align: center">10</td>
      <td style="text-align: left">number of consecutive unstable restarts (less than 1sec interval or custom time via min_uptime) before your app is considered errored and stop being restarted</td>
    </tr>
    <tr>
      <td style="text-align: left">restart_delay</td>
      <td style="text-align: center">number</td>
      <td style="text-align: center">4000</td>
      <td style="text-align: left">time to wait before restarting a crashed app (in milliseconds). defaults to 0.</td>
    </tr>
    <tr>
      <td style="text-align: left">autorestart</td>
      <td style="text-align: center">boolean</td>
      <td style="text-align: center">false</td>
      <td style="text-align: left">true by default. if false, PM2 will not restart your app if it crashes or ends peacefully</td>
    </tr>
    <tr>
      <td style="text-align: left">cron_restart</td>
      <td style="text-align: center">string</td>
      <td style="text-align: center">“1 0 * * *”</td>
      <td style="text-align: left">a cron pattern to restart your app. Application must be running for cron feature to work</td>
    </tr>
    <tr>
      <td style="text-align: left">vizion</td>
      <td style="text-align: center">boolean</td>
      <td style="text-align: center">false</td>
      <td style="text-align: left">true by default. if false, PM2 will start without vizion features (versioning control metadatas)</td>
    </tr>
    <tr>
      <td style="text-align: left">post_update</td>
      <td style="text-align: center">list</td>
      <td style="text-align: center">[“npm install”, “echo launching the app”]</td>
      <td style="text-align: left">a list of commands which will be executed after you perform a Pull/Upgrade operation from Keymetrics dashboard</td>
    </tr>
    <tr>
      <td style="text-align: left">force</td>
      <td style="text-align: center">boolean</td>
      <td style="text-align: center">true</td>
      <td style="text-align: left">defaults to false. if true, you can start the same script several times which is usually not allowed by PM2</td>
    </tr>
  </tbody>
  
</table>
{% endraw %}

> 使用json配置文件的方式，命令行可选参数会无效,


日志文件

```
# 不保存日志 
设置 /dev/null to error_file or out_file  不保存日志文件.

# 同一应用 日志内容放一起
使用 merge_logs: true 禁止日志文件自动加上进程id后缀 (e.g. app-name-ID.log) 
```


### 状态管理

#### 平滑停止

```
# Graceful restart/reload/stop
process.on('SIGINT', function() {
   db.stop(function(err) {
     process.exit(err ? 1 : 0);
   });
});

# 延时 3000ms
pm2 start app.js --kill-timeout 3000

{
  "apps" : [{
    "name"         : "api",
    "script"       : "app.js",
    "kill_timeout" : 3000
  }]
}
```

#### 平滑启动
某些应用运行需要等待DB或者workers建立好链接

```
# 应用发出ready正式更新状态
pm2 start app.js --wait-ready

# 应用链接处理完毕发送ready消息
var http = require('http');
var app = http.createServer(function(req, res) {
  res.writeHead(200);
  res.end('hey');
})
var listener = app.listen(0, function() {
  console.log('Listening on port ' + listener.address().port);
  // Here we send the ready signal to PM2
  process.send('ready');
});

# 默认超时时间为 3000ms
pm2 start app.js --wait-ready --listen-timeout 3000

# json方式指定
{
  "apps" : [{
    "name"         : "api",
    "script"       : "app.js",
    "listen_timeout" : 3000
  }]
}

```

### 环境变量

```
# 1.PM2 默认注入当前shell的环境变量
# 2.PM2注入配置文件中声明的环境变量，重名则覆盖

module.exports = {
  apps : [
      {
        name: "myapp",
        script: "./app.js",
        watch: true,
        env: {
            "PORT": 3000,
            "NODE_ENV": "development"
        },
        env_production: {
            "PORT": 80,
            "NODE_ENV": "production",
        }
      }
  ]
}

# 根据运行环境，选择不同的环境变量
ecosystem.config.js --env production

# 默认  restarting or reloading 不会更新环境变量，如果需要更新可以使用如下方式
pm2 reload ecosystem.json --update-env

# 指定实例个数的环境变量
NODE_APP_INSTANCE 

# 修改环境变量的名称
instance_var: 'INSTANCE_ID',

```



### 应用日志管理
PM2日志可以合并，也可以存放不同的日志文件中。

#### 实时显示日志

```
# 查看logs命令帮助
pm2 logs -h

# 显示所有应用日志
pm2 logs

# 指定正则匹配的应用的日志
pm2 logs /api/

pm2 logs /server_[12]/

# 显示应用api的日志
pm2 logs api

# 显示应用api日志的 X 行
pm2 logs big-api --lines 1000

# 格式化日志显示
pm2 logs --json pm2 logs --format

# 应用启动指定显示方式 
pm2 start --log-type json app.json
pm2 start echo.js --merge-logs --log-date-format="YYYY-MM-DD HH:mm Z"

# 或者配置文件中中指定 "log_type": "json"

# 清除日志 （或者使用 pm2 install pm2-logrotate ）
pm2 flush

# 不记录日志信息
{
  "out_file": "/dev/null",
  "error_file": "/dev/null"
}

# 指定日志类型
pm2 logs main
```

### 更新PM2

```
# 保存当前进程信息
pm2 save

# 安装最新版pm2
npm install pm2 -g

# 在内存中更新pm2
pm2 update
```

### 部署
本地我们可以直接通过 `pm2 start/stop`管理本地应用，通过deploy命令，在本地可以很轻松的将本地的应用部署远程服务器上。

```
{
   "apps" : [{
      "name" : "HTTP-API",
      "script" : "http.js"
   }],
   "deploy" : {
     // "production" is the environment name
     "production" : {
        // 已授权的公钥路径
        "key"  : "/path/to/some.pem",           // 用户名
       "user" : "ubuntu",
       // 主机
       "host" : ["212.83.163.1", "212.83.163.2", "212.83.163.3"],
       // 分支
       "ref"  : "origin/master",
       // 代码库
       "repo" : "git@github.com:Username/repository.git",
       // 远程服务器存放应用的路径
       "path" : "/var/www/my-repository",
       // ssh 配置项 可以是字符串或者数组字符串
       "ssh_options": "StrictHostKeyChecking=no",
       // 安装依赖，可以是命令，多个命令用分号;分割，也可以用本地脚本文件的路径
       "pre-setup" : "apt-get install git",
       // 代码库克隆完成执行 使用方式和 pre-setup一样
       "post-setup": "ls -la",
       // 本地要执行的代码
       "pre-deploy-local" : "echo 'This is a local executed command'"
       // 代码库克隆完成,远程执行
       "post-deploy" : "npm install; grunt dist"
      },
      "dev" : {
       "user" : "ubuntu_dev",
       "host" : ["192.168.0.12"],
       "ref"  : "origin/master",
       "repo" : "git@github.com:Username/repository.git",
       "path" : "/var/www/my-repository",
       "post-deploy" : "npm install; grunt dist"
      }
   }
}

# 本地生成公钥
ssh-keygen -t rsa
ssh-copy-id node@myserver.com

# 远程部署，初始化应用目录
pm2 deploy production setup

# 更新远程版本
pm2 deploy production update

# 版本回退 Revert to -1 deployment
pm2 deploy production revert 1

# 远程执行命令
pm2 deploy production exec "pm2 reload all"

pm2 startOrRestart all.json  

```

[Docker部署](http://pm2.keymetrics.io/docs/usage/docker-pm2-nodejs/)

### 将pm2设置成系统服务

* systemd: Ubuntu >= 16, CentOS >= 7, Arch, Debian >= 7
* upstart: Ubuntu <= 14
* launchd: Darwin, MacOSx
* openrc: Gentoo Linux, Arch Linux
* rcd: FreeBSD
* systemv: Centos 6, Amazon Linux

```
pm2 startup

# 指定具体平台,不加参数会自动检测
pm2 startup [ubuntu | ubuntu14 | ubuntu12 | centos | centos6 | arch | oracle | amazon | macos | darwin | freebsd | systemd | systemv | upstart | launchd | rcd | openrc]

# disable 自动启动
pm2 unstartup

# 保存当前进程列表
pm2 save

# 恢复之前保存的进程列表
pm2 resurrect

```

检测自启服务是否设置好

```
# Check if pm2-<USER> service has been added
$ systemctl list-units
# Check logs
$ journalctl -u pm2-<USER>
# Cat systemd configuration file
$ systemctl cat pm2-<USER>
# Analyze startup
$ systemd-analyze plot > output.svg
```

### 自动重启服务

```
# 当前目录或子目录文件改动自动重启服务
pm2 start app.js --watch

# 不会停止检测文件改动
pm2 stop 0 

# 停止检测文件改动
pm2 stop --watch 0 

# 配置文件 详细配置检测文件路径
{
  "watch": ["server", "client"],
  "ignore_watch" : ["node_modules", "client/img"],
  "watch_options": {
    "followSymlinks": false
  }
}

# 监测cpu及memory
pm2 monit
```

### sourcemap
当使用babeljs,typescript时，有了sourcemap，异常了可以找到具体错误信息。

```
# 命令行指定 
pm2 start app.js --source-map-support
pm2 start app.js --disable-source-map

# 配置文件指定
{
  "apps" : [{
    "name"              : "my-app",
    "script"            : "script.js",
    "source_map_support": true
  }]
}
```

### 静态服务
pm2 2.4.0 后可使用静态服务命令

```
# 命令行方式
pm2 serve <path> <port>

# 或者配置文件
{
  "script": "serve",
  "env": {
    "PM2_SERVE_PATH": '.',
    "PM2_SERVE_PORT": 8080
  }
}
```

### 其他
* [进程性能指标](http://pm2.keymetrics.io/docs/usage/process-metrics/#counter-sequential-value-change)
* [非root使用authbind绑定80端口](http://pm2.keymetrics.io/docs/usage/specifics/)
* [pm2 api](http://pm2.keymetrics.io/docs/usage/pm2-api/)
* [云服务使用pm2](http://pm2.keymetrics.io/docs/usage/use-pm2-with-cloud-providers/)
* [pm2模块](http://pm2.keymetrics.io/docs/advanced/pm2-module-system/)