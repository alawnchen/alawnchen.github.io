---
title: Linux上用supervisor守护进程
date: 2018-06-02 07:53:00
tags: 
  - supervisor
  - 守护进程
categories:
  - 技术
url: run-daemon-process-with-supervisor-on-linux
---

> 很多时候，需要在后台长期、随机启动运行一些，而且还得保证服务器稳定和可靠。所以，就出现了很多进程管理系统，而supervisor就是众多软件中的佼佼者！

<!--more-->


## 环境说明

我的Linux系统是centos，是最新版本的7.4，64bit，系统内核版本（可通过uname -a命令查看系统内核版本）3.10.0-693.2.2.el7.x86_64。

## 安装supervisor

按照官网说明，安装supervisor可以通过四种方式安装。简单说明一下，请客官们自己选择最适合自己的安装方式！

### 联网使用setuptools安装

联网情况下使用setuptools安装。这里用到的是easy_install。easy_install
是一个基于setuptools的工具，帮助我们自动下载、编译、安装和管理python包。当然了，安装了setuptools和联网都是必备条件！

```
easy_install supervisor
```

### 联网不用setuptools安装

首先需要下载supervisor的源码包。supervisor在pypi上的托管地址是pypi.python.org/pypi/supervisor
。当然了，也可以从其他渠道获取supervisor源码包。下载完成后，解压安装。

```
wget https://files.pythonhosted.org/packages/44/60/698e54b4a4a9b956b2d709b4b7b676119c833d811d53ee2500f1b5e96dc3/supervisor-3.3.4.tar.gz
tar xzf supervisor-3.3.4.tar.gz
cd supervisor*
python setup.py install
```

### 无网络情况下安装

有些Linux发行版会自带supervisor分发包，虽然这些supervisor的版本可能远低于官网的。不过，聊胜于无，也是没办法的情况下的方式。首先需要确定的是有没有携带supervisor包，直接启动使用即可！

```
yum info supervisor
```

如果是ubuntu

```
apt-cache show supervisor
```

如果确定有，可直接通过supervisord尝试启动！

### pip安装方式

pip是一个python的软件包管理系统，使用起来是非常666的！

```
pip install supervisor
```

## 生成配置

supervisor提供了一个很好的工具echo_supervisord_conf来初始化配置。

```
echo_supervisord_conf > /etc/supervisord.conf
```

如果执行上面的命令没有反应或者报错，请尝试使用root账号或者sudo权限尝试！如果使用了root账号或者sudo权限还是不行，可更换配置文件目录！

## 组件

supervisor提供了supervisord和supervisorctl两个可执行程序。supervisord负责管理我们添加上去的程序。按其说法是，supervisord是把我们添加上去的程序以子进程形式运行管理的。而supervisorctl负责调度supervisord。

启动supervisord

```
supervisord
```

supervisord的配置默认存放路径是/etc/supervisord.conf。如果在生成配置的时候，修改了配置路径，就需要通过-c命令行参数指定配置文件路径。

```
supervisord -c supervisord.conf
```

supervisord命令行参数
参数参数说明-c FILE, --configuration=FILE指定supervisor配置文件路径，FILE是配置文件绝对路径-n,
--nodaemon前台运行，不以守护方式运行-h, --help显示帮助信息-u USER,
--user=USER指定运行用户，Unix用户名或者用户ID-m OCTAL, --umask=OCTAL掩码-d PATH,
--directory=PATH当supervisord以daemon（守护进程）方式运行时候，在守护进程之前会切换到这个目录-l FILE,
--logfile=FILE指定supervisord活动日志的绝对路径-y BYTES,
--logfile_maxbytes=BYTES指定日志切割大小，可携带单位，例如1MB或者1GB-y NUM,
--logfile_backups=NUM保留日志数量，多出的部分将会被清除-e LEVEL, --loglevel=LEVEL日志级别，可选的有trace,
debug, info, warn, error以及critical-j FILE,
--pidfile=FILE指定PID文件绝对路径，daemon运行程序时候会自动创建-i STRING,
--identifier=STRING指定一个标识符，标注实例-q PATH,
--childlogdir=PATH指定子进程的默认日志目录，目录必须已经存在-k, --nocleanup终端退出时候，不退出supervisord-a
NUM, --minfds=NUM指定最小的文件描述符数量-t, --strip_ansi从所有子日志进程中剥离ANSI转义序列-v,
--version输出supervisord版本号--profile_options=LIST性能分析器，可支持的有cumulative, calls,
callers，可同时支持多个，用半角逗号隔开--minprocs=NUM最小的进程数量

supervisorctl使用
supervisorctl执行的命令都是一次性的，没有可持续性。可以把supervisorctl当成类似于systemctl的一个工具，但又不尽相同。

supervisorctl命令行参数
参数参数说明-c, --configuration指定配置文件路径，后面必须加上文件的绝对路径-h, --help显示帮助信息-i,
--interactive交互模式-s, --serverurl URL指定supervisord服务器地址，默认是
http://localhost:9001-u, --username指定supervisord用户名-p,
--password指定supervisord用户密码-r, --history-file保存readline历史文件（readline可用的前提下）

上面的配置是临时性地指定supervisord配置，可以用在一些临时情况下。例如，想尝试用另一个配置运行supervisord测试。

supervisorct子命令（action）
命令命令说明help显示所有可用的子命令help 显示某一个子命令的帮助信息add  [...]更新某一个进程/分组的配置文件remove
 [...]移除某一个进程/分组的配置文件update重载配置文件，然后增删变更的配置（可能会重启程序）clear 清除某个进程的日志clear
 清除多个进程的日志clear all清除所有进程的日志fg 前台模式连接某个进程，按CTRL+C退出前台模式pid获取supervisord的进程IDpid
获取某个子进程的进程IDpid all获取所有子进程的进程ID，一行一个reread重载daemon配置文件，不会添加/移除配置信息（也不会重启）restart
重启某个进程，不会自动读取最新配置，必须先通过update/reread处理restart :*重启某个分组的所有进程restart
 重启多个进程，或者多个分组对应的所有进程restart all重启全部进程signal信号start 启动某个进程start
:*启动某个分组的所有进程start  启动多个进程，或者多个分组对应的所有进程start all启动所有进程status获取所有进程信息status
获取某个进程信息status  获取多个进程信息stop 停止某个进程stop :*停止某个分组的所有进程stop  停止多个进程stop
all停止所有进程tail [-f]  [stdout/stderr]实时输出某个进程日志

## 守护supervisord进程

官方提供了守护进程例子托管在github上面，地址是https://github.com/Supervisor/initscripts

```
[Unit]
Description=Supervisor daemon
[Service]
Type=forking
ExecStart=/usr/bin/supervisord -c /etc/supervisord.conf
ExecStop=/usr/bin/supervisorctl $OPTIONS shutdown
ExecReload=/usr/bin/supervisorctl $OPTIONS reload
KillMode=process
Restart=on-failure
RestartSec=42s
[Install]
WantedBy=multi-user.target
```

上面的是centos的systemctl守护进程服务脚本。

## supervisord配置文件说明

supervisor配置文件会分成几个部分，每个部分都是用[]区分

```
[unix_http_server]
file=/tmp/supervisor.sock   ;UNIX socket 文件，supervisorctl 会使用
;chmod=0700                 ;socket文件的mode，默认是0700
;chown=nobody:nogroup       ;socket文件的owner，格式：uid:gid

;[inet_http_server]         ;HTTP服务器，提供web管理界面
;port=127.0.0.1:9001        ;Web管理后台运行的IP和端口，如果开放到公网，需要注意安全性
;username=user              ;登录管理后台的用户名
;password=123               ;登录管理后台的密码

[supervisord]
logfile=/tmp/supervisord.log ;日志文件，默认是 $CWD/supervisord.log
logfile_maxbytes=50MB        ;日志文件大小，超出会rotate，默认 50MB，如果设成0，表示不限制大小
logfile_backups=10           ;日志文件保留备份数量默认10，设为0表示不备份
loglevel=info                ;日志级别，默认info，其它: debug,warn,trace
pidfile=/tmp/supervisord.pid ;pid 文件
nodaemon=false               ;是否在前台启动，默认是false，即以 daemon 的方式启动
minfds=1024                  ;可以打开的文件描述符的最小值，默认 1024
minprocs=200                 ;可以打开的进程数的最小值，默认 200

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ;通过UNIX socket连接supervisord，路径与unix_http_server部分的file一致
;serverurl=http://127.0.0.1:9001 ; 通过HTTP的方式连接supervisord

; [program:xx]是被管理的进程配置参数，xx是进程的名称
[program:xx]
command=/opt/apache-tomcat-8.0.35/bin/catalina.sh run  ; 程序启动命令
autostart=true       ; 在supervisord启动的时候也自动启动
startsecs=10         ; 启动10秒后没有异常退出，就表示进程正常启动了，默认为1秒
autorestart=true     ; 程序退出后自动重启,可选值：[unexpected,true,false]，默认为unexpected，表示进程意外杀死后才重启
startretries=3       ; 启动失败自动重试次数，默认是3
user=tomcat          ; 用哪个用户启动进程，默认是root
priority=999         ; 进程启动优先级，默认999，值小的优先启动
redirect_stderr=true ; 把stderr重定向到stdout，默认false
stdout_logfile_maxbytes=20MB  ; stdout 日志文件大小，默认50MB
stdout_logfile_backups = 20   ; stdout 日志文件备份数，默认是10
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
stdout_logfile=/opt/apache-tomcat-8.0.35/logs/catalina.out
stopasgroup=false     ;默认为false,进程被杀死时，是否向这个进程组发送stop信号，包括子进程
killasgroup=false     ;默认为false，向进程组发送kill信号，包括子进程

;[eventlistener:theeventlistenername] ;侦听器
;command=/bin/eventlistener    ; 侦听器路径
;process_name=%(program_name)s ; 进程名称
;numprocs=1                    ; 进程初始化数量
;events=EVENT                  ; 订阅的事件类型
;buffer_size=10                ; 事件缓冲大小
;directory=/tmp                ; exec之前要打开的目录
;umask=022                     ; 进程掩码
;priority=-1                   ; 启动优先权
;autostart=true                ; 是否跟随supervisord启动
;startsecs=1                   ; # 判断启动成功需要的侦听时间长度
;startretries=3                ; 最大尝试启动次数
;autorestart=unexpected        ; 自动重启情况（正常退出也会重启）
;exitcodes=0,2                 ; 'expected'（异常）退出码
;stopsignal=QUIT               ; 停止信号
;stopwaitsecs=10               ; 停止前等待时长
;stopasgroup=false             ; 发送停止信号给进程组
;killasgroup=false             ; 停止进程组信号
;user=chrism                   ; 运行侦听器用户
;redirect_stderr=false         ; redirect_stderr=true时不使用侦听器
;stdout_logfile=/a/path        ; stdout日志路径，默认AUTO
;stdout_logfile_maxbytes=1MB   ;  logfile切割大小，默认最大50MB
;stdout_logfile_backups=10     ;  stdout保留日志数量
;stdout_events_enabled=false   ; stdout写日志时候广播
;stderr_logfile=/a/path        ; stderr日志路径，默认AUTO
;stderr_logfile_maxbytes=1MB   ; logfile切割大小，默认最大50MB
;stderr_logfile_backups=10     ; stderr保留日志数量
;stderr_events_enabled=false   ; stderr写入时广播事件
;environment=A="1",B="2"       ; 进程环境附加参数
;serverurl=AUTO                ; 覆盖前面的serverurl

;[group:thegroupname] ;分组
;programs=progname1,progname2  ; 分组里面的程序名，用半角逗号隔开
;priority=999                  ; 优先权限

;包含其它配置文件,一般情况下，我们会新建一个目录来专门存放服务的配置文件，然后在此文件中将其include包含进来。
[include]
files = relative/directory/*.ini    ;可以指定一个或多个以.ini结束的配置文件
```