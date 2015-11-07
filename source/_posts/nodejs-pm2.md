title: PM2（Node.js 进程守护工具）
date: 2015-10-26 11:17:08
tags: Node.js
categories: Node.js
---

PM2 是一个 Node.js 进程守护工具，可以让我们很方便的对 Node.js 程序进行管理，类似的工具还有 [forever](https://github.com/foreverjs/forever)。

### 基本的用法

####  启动进程

`pm2 start xx.js`
指定启动的 js 文件

`pm2 start xxx.json`
指定启动的配置文件，配置文件的格式

```
{
  "apps" : [{
    "name"        : "worker-app",
    "script"      : "worker.js",
    "args"        : ["--toto=heya coco", "-d", "1"],
    "watch"       : true,
    "node_args"   : "--harmony",
    "merge_logs"  : true,
    "cwd"         : "/this/is/a/path/to/start/script",
    "env": {
        "NODE_ENV": "production",
        "AWESOME_SERVICE_API_TOKEN": "xxx"
    }
  },{
    "name"       : "api-app",
    "script"     : "api.js",
    "instances"  : 4,
    "exec_mode"  : "cluster_mode",
    "error_file" : "./examples/child-err.log",
    "out_file"   : "./examples/child-out.log",
    "pid_file"   : "./examples/child.pid"
  }]
}
```

详细的配置项看[这里](http://pm2.keymetrics.io/docs/usage/application-declaration/)。

#### 重新加载

`pm2 reload [name|id]`

`pm2 restart [name|id]`

#### 停止

`pm2 stop [name|id]`
停止程序运行

`pm2 delete [name|id]`
停止程序并删除相关信息

#### 查看运行状态及相关信息

`pm2 list`
`pm2 info [name|id]`
`pm2 desc [name|id]`
`pm2 monit [name|id]`

上面的命令都会显示程序的运行状态或展示相关信息。

#### 设置开机启动项

1. `pm2 dump [name|id]`
保存程序运行的相关信息
2. `pm2 startup [platform]`
生成设置开机启动项命令，类型这样

        $ pm2 startup
        [PM2] You have to run this command as root. Execute the following command:
        sudo su -c "env PATH=$PATH:/Users/xxx/.nvm/versions/io.js/v2.5.0/bin pm2 startup darwin -u xxx"


### 扩展插件

#### pm2-logrotate

自动管理日志文件（[项目地址](https://github.com/pm2-hive/pm2-logrotate)）

##### 安装

`pm2 install pm2-logrotate`

**注意** 如果 `npm` 的版本是 `v2.7.4` ，会安装失败（Github 上的 [issue](https://github.com/npm/npm/issues/7984)），需更新 `npm`。

##### 配置

- max_size（默认 10MB）：当日志文件超过该值指定的大小时，就会切割日志文件。还可以设置为其他的单位，例如：1G、1M、1K。
- interval（默认 1）
- interval_unit（默认 'DD'）：interval 和 interval_unit 是一起配合使用的，例如 interval=3、interval_unit='DD'，表示日志文件每三天切割一次。
- retain（默认 'all'）：保留的日志文件数

##### 设置

`pm2 set pm2-logrotate:<param> <value>`
配置项的值都是通过这样的形式设置，例如：
`pm2 set pm2-logrotate:max_size 1K`
设置 max_size 为 1K

`pm2 get`
查看设置的配置项值
