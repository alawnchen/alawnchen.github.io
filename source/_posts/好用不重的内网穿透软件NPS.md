---
title: 好用不重的内网穿透软件NPS
date: 2020-03-04 08:16:00
tags: 
  - nps
  - 内网穿透
categories:
  - 技术
url: hao-yong-bu-zhong-de-nei-wang-chuan-tou-ruan-jian-nps
---

> 内网穿透用过几个了，有用过群晖自带，用过FRP，用过ngrok，用过阿里云DDNS，还尝试用过LEDE自带的Kool
DDNS。但是说到最满意的，还是当下使用的NPS。NPS的特点嘛，简单、好用、硬件要求低、轻量，还有配置简单和跨平台性极好。

<!--more-->

## 内网穿透干嘛

干啥用呢，说几个使用场景吧。第一，公司电脑上有工作需要的文件，家里电脑没有，突然需要工作需要的文件，可以通过内网穿透实现文件访问获取到需要的文件；第二，公司电脑上有特定的程序，在家里电脑安装不了，突然需要使用那个程序，可以通过内网穿透实现远程控制公司电脑使用程序；第三，公司的电脑需要开机，那么可以通过内网穿透远程访问路由器（作为堡垒机），利用WOL（局域网唤醒）唤醒目标电脑；第四，想玩的游戏的服务器架设在国内，国外玩起来很卡，那么可以通过内网穿透代理UDP，实现游戏加速……

## 服务端

NPS服务端的安装非常简单，这里给出简单的步骤，同时给出一个简单的配置。我的服务器是Linux 64Bit，其他的平台类似。当前，NPS最新版本是0.26.4。

```
mkdir -p /data/nps #创建目录
wget https://github.com/ehang-io/nps/releases/download/v0.26.4/linux_amd64_server.tar.gz -O /data/linux_amd64_server.tar.gz #下载文件
tar xzf /data/linux_amd64_server.tar.gz -C /data/nps/ #解压文件
```

编辑配置/data/nps/conf/nps.conf
文件，修改成如下（public_vkey、web_username和web_password分别代表验证密钥、web管理需要的用户名和密码，自行修改）：

```
appname = nps
#Boot mode(dev|pro)
runmode = pro

#HTTP(S) proxy port, no startup if empty
http_proxy_ip=0.0.0.0
http_proxy_port=20080
https_proxy_port=20443
https_just_proxy=true
#default https certificate setting
https_default_cert_file=conf/server.pem
https_default_key_file=conf/server.key

##bridge
bridge_type=tcp
bridge_port=8024
bridge_ip=0.0.0.0

# Public password, which clients can use to connect to the server
# After the connection, the server will be able to open relevant ports and parse related domain names according to its own configuration file.
public_vkey=leochan1234556789

#Traffic data persistence interval(minute)
#Ignorance means no persistence
#flow_store_interval=1

# log level LevelEmergency->0  LevelAlert->1 LevelCritical->2 LevelError->3 LevelWarning->4 LevelNotice->5 LevelInformational->6 LevelDebug->7
log_level=7
#log_path=nps.log

#Whether to restrict IP access, true or false or ignore
#ip_limit=true

#p2p
#p2p_ip=127.0.0.1
#p2p_port=6000

#web
#web_host=host
web_username=yonghuming
web_password=mima123456
web_port = 8080
web_ip=0.0.0.0
web_base_url=
web_open_ssl=false
web_cert_file=conf/server.pem
web_key_file=conf/server.key
# if web under proxy use sub path. like http://host/nps need this.
#web_base_url=/nps

#Web API unauthenticated IP address(the len of auth_crypt_key must be 16)
#Remove comments if needed
#auth_key=test
auth_crypt_key =1234567812345678

#allow_ports=9001-9009,10001,11000-12000

#Web management multi-user login
allow_user_login=false
allow_user_register=false
allow_user_change_username=false


#extension
allow_flow_limit=false
allow_rate_limit=false
allow_tunnel_num_limit=false
allow_local_proxy=false
allow_connection_num_limit=false
allow_multi_ip=false
system_info_display=false

#c
http_cache=true
http_cache_length=100

#get origin ip
http_add_origin_header=false

#pprof debug options
#pprof_ip=0.0.0.0
#pprof_port=9999
```


继续执行以下命令完成安装并启动！！

```
cd /data/nps #切换目录
./nps install #开始安装
nps start #启动引擎
```


## 反向代理

反向代理，我用的是Caddy，因为简单高效，最重要的是不用管证书，可以自动获取更新SSL证书。Caddy的2.0版本已经出来，但还处于beta版本，所以我用的是1.0系列版本。Caddy自带启动脚本，我用的服务器系统是centos
7.4 64bit，所以，我用到的启动脚本是init/linux-systemd/caddy.service。根据启动脚本的内容，得知需要把caddy可执行文件放到/usr/local/bin目录，并且确定配置文件是/etc/caddy/Caddyfile。

```
mkdir -p /data/caddy #建立基地
wget https://github.com/caddyserver/caddy/releases/download/v1.0.3/caddy_v1.0.3_linux_amd64.tar.gz -O /data/caddy/caddy_v1.0.3_linux_amd64.tar.gz #运货到基地
cp /data/caddy/caddy /usr/local/bin/ #拷贝可执行文件到目标目录
chmod a+x /usr/local/bin/caddy #更改权限
cat /data/caddy/init/linux-systemd/caddy.service > /etc/systemd/system/caddy.service #搬动启动脚本内容
sed -i  ’s/www-data/root/g’ /etc/systemd/system/caddy.service #替换caddy运行用户，懒得去处理权限问题
chmod 755 /etc/systemd/system/caddy.service #更改脚本文件权限
systemctl enable caddy.service #自启动
mkdir -p /etc/ssl/caddy #建立caddy证书存取目录
mkdir -p /var/log/caddy #建立日志目录
mkdir -p /etc/caddy #建立以下配置存放目录
touch /etc/caddy/Caddyfile #建立配置文件
```

修改/etc/caddy/Caddyfile文件并保存，内容如下（nps.domain.tls是绑定的域名，10000@163.com
是邮箱地址，请按照自己的实际内容修改）：

```
nps.domain.tls {                                                                                                                           
        gzip                                                                                                                               
        redir  301 {                                                                                                                       
                if {>X-Forwarded-Proto} is http                                                                                            
                / https://nps.domain.tls{uri}                                                                                              
        }
        tls 10000@163.com                                                    
        log /var/log/caddy/nps.domain.tls.log {                                                                                            
                rotate_size 5                                                                                                              
                rotate_age 5                                                                                                               
                rotate_keep 2                                                                                                              
                rotate_compress                                                                                                            
        }                                                                                                                                  
        proxy / 127.0.0.1:8080 {                                                                                                           
                transparent                                                                                                                
        }                                                                                                                                  
}
```

## 启动服务端

```
echo $PATH #查看以下/usr/local/bin是否包含在环境变量里，如果没有匹配，请执行下三步的环境变量配置和环境变量生效，否则跳过；
echo ‘PATH=$PATH:/usr/local/bin’ >> ~/.bash_profile #环境变量配置
echo ‘export PATH’ >> ~/.bash_profile #环境变量配置
source ~/.bash_profile #环境变量生效
systemctl start caddy.service #启动caddy
```

## caddy网页端管理

访问https://nps.domain.tls
，输入上面配置文件里面的web_username和web_password登录。点击左边菜单“客户端”，点击“新增”按钮新增客户端。按照下图填写表单，填写好了，点击“新增”按钮保存。



















客户端
大部分的客户端安装都比较简单，也基本是一键安装。这里还是以Linux 64Bit为例安装客户端。

mkdir -p /data/npc #找块地
wget https://github.com/ehang-io/nps/releases/download/v0.26.4/linux_amd64_client.tar.gz -O /data/linux_amd64_client.tar.gz #拿到货
tar xzf /data/linux_amd64_client.tar.gz -C /data/npc/ #卸货


编辑配置/data/npc/conf/npc.conf，内容如下（vkey是添加客户端时候需要用到的，详看caddy网页端管理那一步的图片，8.8.8.8换成真实的服务器IP地址）

[common]
server_addr=8.8.8.8:8024
conn_type=tcp
vkey=leochan.me
auto_reconnection=true
max_conn=1000
flow_limit=1000
rate_limit=1000
crypt=true
compress=true


执行以下命令，完成安装并启动客户端

cd /data/npc #找到解压目录
./npc install -config=/data/npc/conf/npc.conf #执行安装
npc start #启动客户端


域名解析
假设，本地局域网有一台机器，局域网IP是192.168.2.50，服务绑定的端口号是5000，绑定的域名是file.domain.tls。那么，请按照下图配置：



客户端ID这样查看

客户端ID















































配置好之后，我们回到反向代理那一步，处理一下域名。编辑/etc/caddy/Caddyfile，追加如下内容：

file.domain.tls {
        gzip
        redir  301 {
                if {>X-Forwarded-Proto} is http
                / https://{host}{uri}
        }
        tls 10000@163.com
        log /var/log/caddy/nps.log {
                rotate_size 5
                rotate_age 5
                rotate_keep 2
                rotate_compress
        }
        proxy / 127.0.0.1:20080 {
                transparent
        }
}


重启一下caddy，收工了！！

后话
需要开辟另一篇来做以下nps的其他代理的介绍！！