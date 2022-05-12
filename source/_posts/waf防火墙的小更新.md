---
title: waf防火墙的小更新
date: 2018-09-12 08:10:00
tags: 
  - waf
  - lua
  - 防火墙
categories:
  - 技术
url: the-modification-of-waf-firewall
---

> lua，很多人应该都听过。据说高性能，据说很简单，据说还可以与nginx配合……因为着很多优点，很多云服务商都提供了基于lua开发的waf防火墙，美其名曰云waf。从这也看出waf应该是很有成效的，阿里云就提供了云waf，用于防御DDOS与CC攻击。所以，我也赶了赶时髦，部署waf到服务器上面。

<!--more-->

## 一点絮叨

很早就开始用waf，用的是ngx_lua_waf，是某位大牛开源出来的。地址是https://github.com/loveshell/ngx_lua_waf
，感兴趣的大牛、中牛、年轻人都可以去看看。用的过程遇到过一些问题，因为长期的忙忙忙，所以，也没什么时间去多看看。最近一个项目因为要求比较严，所以，我又翻了翻ngx_lua_waf的github页面。这才发现了虽然项目已经停止更新有一段时间，还有不少的Pull
requests，甚至于最近的Pull requests是9天前的。把Pull requests筛选了一遍，就有了这次的小更新！

## waf.lua

```
local content_length=tonumber(ngx.req.get_headers()['content-length'])
local method=ngx.req.get_method()
--ngx_lua如果是0.9.2以上版本，建议正则过滤函数改为ngx.re.find，匹配效率会提高三倍左右。
local ngxmatch=ngx.re.match
--local ngxmatch=ngx.re.find
if whiteip() then
elseif whiteurl() then
elseif blockip() then
--检测攻击ip是否被拦截。
elseif denyhackip(1) then
elseif denycc() then
elseif ngx.var.http_Acunetix_Aspect then
    ngx.exit(444)
elseif ngx.var.http_X_Scan_Memo then
    ngx.exit(444)
elseif ua() then
elseif url() then
elseif args() then
elseif cookie() then
elseif PostCheck then
    if method=="POST" then
            local boundary = get_boundary()
	    if boundary then
	    local len = string.len
            local sock, err = ngx.req.socket()
    	    if not sock then
					return
            end
	    ngx.req.init_body(128 * 1024)
            sock:settimeout(0)
	    local content_length = nil
    	    content_length=tonumber(ngx.req.get_headers()['content-length'])
    	    local chunk_size = 4096
            if content_length < chunk_size then
					chunk_size = content_length
	    end
            local size = 0
	    while size < content_length do
		local data, err, partial = sock:receive(chunk_size)
		data = data or partial
		if not data then
			return
		end
		ngx.req.append_body(data)
        	if body(data) then
	   	        return true
    	    	end
		size = size + len(data)
		local m = ngxmatch(data,[[Content-Disposition: form-data;(.+)filename="(.+)\.(.*)"]],'ijo')
        	if m then
            		fileExtCheck(m[3])
            		filetranslate = true
        	else
            		if ngxmatch(data,"Content-Disposition:",'isjo') then
                		filetranslate = false
            		end
            		if filetranslate==false then
            			if body(data) then
                    			return true
                		end
            		end
        	end
		local less = content_length - size
		if less < chunk_size then
			chunk_size = less
		end
	 end
	 ngx.req.finish_body()
    else
			ngx.req.read_body()
			local args = ngx.req.get_post_args()
			if not args then
				return
			end
			for key, val in pairs(args) do
				if type(val) == "table" then
					if type(val[1]) == "boolean" then
						return
					end
					data=table.concat(val, ", ")
				else
					data=val
				end
				if data and type(data) ~= "boolean" and body(data) then
                			body(key)
				end
			end
		end
		end

else
    return
end
```

## socket.lua

```
-- Copyright (C) 2013-2014 Jiale Zhi (calio), CloudFlare Inc.
--require "luacov"

local concat                = table.concat
local tcp                   = ngx.socket.tcp
local udp                   = ngx.socket.udp
local timer_at              = ngx.timer.at
local ngx_log               = ngx.log
local ngx_sleep             = ngx.sleep
local type                  = type
local pairs                 = pairs
local tostring              = tostring
local debug                 = ngx.config.debug

local DEBUG                 = ngx.DEBUG
local CRIT                  = ngx.CRIT

local MAX_PORT              = 65535


-- table.new(narr, nrec)
local succ, new_tab = pcall(require, "table.new")
if not succ then
    new_tab = function () return {} end
end

local _M = new_tab(0, 5)

local is_exiting

if not ngx.config or not ngx.config.ngx_lua_version
    or ngx.config.ngx_lua_version < 9003 then

    is_exiting = function() return false end

    ngx_log(CRIT, "We strongly recommend you to update your ngx_lua module to "
            .. "0.9.3 or above. lua-resty-logger-socket will lose some log "
            .. "messages when Nginx reloads if it works with ngx_lua module "
            .. "below 0.9.3")
else
    is_exiting = ngx.worker.exiting
end


_M._VERSION = '0.03'

-- user config
local flush_limit           = 4096         -- 4KB
local drop_limit            = 1048576      -- 1MB
local timeout               = 1000         -- 1 sec
local host
local port
local ssl                   = false
local ssl_verify            = true
local sni_host
local path
local max_buffer_reuse      = 10000        -- reuse buffer for at most 10000
                                           -- times
local periodic_flush        = nil
local need_periodic_flush   = nil
local sock_type             = 'tcp'

-- internal variables
local buffer_size           = 0
-- 2nd level buffer, it stores logs ready to be sent out
local send_buffer           = ""
-- 1st level buffer, it stores incoming logs
local log_buffer_data       = new_tab(20000, 0)
-- number of log lines in current 1st level buffer, starts from 0
local log_buffer_index      = 0

local last_error

local connecting
local connected
local exiting
local retry_connect         = 0
local retry_send            = 0
local max_retry_times       = 3
local retry_interval        = 100         -- 0.1s
local pool_size             = 10
local flushing
local logger_initted
local counter               = 0
local ssl_session

local function _write_error(msg)
    last_error = msg
end

local function _do_connect()
    local ok, err, sock

    if not connected then
        if (sock_type == 'udp') then
            sock, err = udp()
        else
            sock, err = tcp()
        end

        if not sock then
            _write_error(err)
            return nil, err
        end

        sock:settimeout(timeout)
    end

    -- "host"/"port" and "path" have already been checked in init()
    if host and port then
        if (sock_type == 'udp') then
            ok, err = sock:setpeername(host, port)
        else
            ok, err = sock:connect(host, port)
        end
    elseif path then
        ok, err = sock:connect("unix:" .. path)
    end

    if not ok then
        return nil, err
    end

    return sock
end

local function _do_handshake(sock)
    if not ssl then
        return sock
    end

    local session, err = sock:sslhandshake(ssl_session, sni_host or host,
                                           ssl_verify)
    if not session then
        return nil, err
    end

    ssl_session = session
    return sock
end

local function _connect()
    local err, sock

    if connecting then
        if debug then
            ngx_log(DEBUG, "previous connection not finished")
        end
        return nil, "previous connection not finished"
    end

    connected = false
    connecting = true

    retry_connect = 0

    while retry_connect <= max_retry_times do
        sock, err = _do_connect()

        if sock then
            sock, err = _do_handshake(sock)
            if sock then
                connected = true
                break
            end
        end

        if debug then
            ngx_log(DEBUG, "reconnect to the log server: ", err)
        end

        -- ngx.sleep time is in seconds
        if not exiting then
            ngx_sleep(retry_interval / 1000)
        end

        retry_connect = retry_connect + 1
    end

    connecting = false
    if not connected then
        return nil, "try to connect to the log server failed after "
                    .. max_retry_times .. " retries: " .. err
    end

    return sock
end

local function _prepare_stream_buffer()
    local packet = concat(log_buffer_data, "", 1, log_buffer_index)
    send_buffer = send_buffer .. packet

    log_buffer_index = 0
    counter = counter + 1
    if counter > max_buffer_reuse then
        log_buffer_data = new_tab(20000, 0)
        counter = 0
        if debug then
            ngx_log(DEBUG, "log buffer reuse limit (" .. max_buffer_reuse
                    .. ") reached, create a new \"log_buffer_data\"")
        end
    end
end

local function _do_flush()
    local ok, err, sock, bytes
    local packet = send_buffer

    sock, err = _connect()
    if not sock then
        return nil, err
    end

    bytes, err = sock:send(packet)
    if not bytes then
        -- "sock:send" always closes current connection on error
        return nil, err
    end

    if debug then
        ngx.update_time()
        ngx_log(DEBUG, ngx.now(), ":log flush:" .. bytes .. ":" .. packet)
    end

    if (sock_type ~= 'udp') then
        ok, err = sock:setkeepalive(0, pool_size)
        if not ok then
            return nil, err
        end
    end

    return bytes
end

local function _need_flush()
    if buffer_size > 0 then
        return true
    end

    return false
end

local function _flush_lock()
    if not flushing then
        if debug then
            ngx_log(DEBUG, "flush lock acquired")
        end
        flushing = true
        return true
    end
    return false
end

local function _flush_unlock()
    if debug then
        ngx_log(DEBUG, "flush lock released")
    end
    flushing = false
end

local function _flush()
    local err

    -- pre check
    if not _flush_lock() then
        if debug then
            ngx_log(DEBUG, "previous flush not finished")
        end
        -- do this later
        return true
    end

    if not _need_flush() then
        if debug then
            ngx_log(DEBUG, "no need to flush:", log_buffer_index)
        end
        _flush_unlock()
        return true
    end

    -- start flushing
    retry_send = 0
    if debug then
        ngx_log(DEBUG, "start flushing")
    end

    local bytes
    while retry_send <= max_retry_times do
        if log_buffer_index > 0 then
            _prepare_stream_buffer()
        end

        bytes, err = _do_flush()

        if bytes then
            break
        end

        if debug then
            ngx_log(DEBUG, "resend log messages to the log server: ", err)
        end

        -- ngx.sleep time is in seconds
        if not exiting then
            ngx_sleep(retry_interval / 1000)
        end

        retry_send = retry_send + 1
    end

    _flush_unlock()

    if not bytes then
        local err_msg = "try to send log messages to the log server "
                        .. "failed after " .. max_retry_times .. " retries: "
                        .. err
        _write_error(err_msg)
        return nil, err_msg
    else
        if debug then
            ngx_log(DEBUG, "send " .. bytes .. " bytes")
        end
    end

    buffer_size = buffer_size - #send_buffer
    send_buffer = ""

    return bytes
end

local function _periodic_flush(premature)
    if premature then
        exiting = true
    end

    if need_periodic_flush or exiting then
        -- no regular flush happened after periodic flush timer had been set
        if debug then
            ngx_log(DEBUG, "performing periodic flush")
        end
        _flush()
    else
        if debug then
            ngx_log(DEBUG, "no need to perform periodic flush: regular flush "
                    .. "happened before")
        end
        need_periodic_flush = true
    end

    timer_at(periodic_flush, _periodic_flush)
end

local function _flush_buffer()
    local ok, err = timer_at(0, _flush)

    need_periodic_flush = false

    if not ok then
        _write_error(err)
        return nil, err
    end
end

local function _write_buffer(msg, len)
    log_buffer_index = log_buffer_index + 1
    log_buffer_data[log_buffer_index] = msg

    buffer_size = buffer_size + len


    return buffer_size
end

function _M.init(user_config)
    if (type(user_config) ~= "table") then
        return nil, "user_config must be a table"
    end

    for k, v in pairs(user_config) do
        if k == "host" then
            if type(v) ~= "string" then
                return nil, '"host" must be a string'
            end
            host = v
        elseif k == "port" then
            if type(v) ~= "number" then
                return nil, '"port" must be a number'
            end
            if v < 0 or v > MAX_PORT then
                return nil, ('"port" out of range 0~%s'):format(MAX_PORT)
            end
            port = v
        elseif k == "path" then
            if type(v) ~= "string" then
                return nil, '"path" must be a string'
            end
            path = v
        elseif k == "sock_type" then
            if type(v) ~= "string" then
                return nil, '"sock_type" must be a string'
            end
            if v ~= "tcp" and v ~= "udp" then
                return nil, '"sock_type" must be "tcp" or "udp"'
            end
            sock_type = v
        elseif k == "flush_limit" then
            if type(v) ~= "number" or v < 0 then
                return nil, 'invalid "flush_limit"'
            end
            flush_limit = v
        elseif k == "drop_limit" then
            if type(v) ~= "number" or v < 0 then
                return nil, 'invalid "drop_limit"'
            end
            drop_limit = v
        elseif k == "timeout" then
            if type(v) ~= "number" or v < 0 then
                return nil, 'invalid "timeout"'
            end
            timeout = v
        elseif k == "max_retry_times" then
            if type(v) ~= "number" or v < 0 then
                return nil, 'invalid "max_retry_times"'
            end
            max_retry_times = v
        elseif k == "retry_interval" then
            if type(v) ~= "number" or v < 0 then
                return nil, 'invalid "retry_interval"'
            end
            -- ngx.sleep time is in seconds
            retry_interval = v
        elseif k == "pool_size" then
            if type(v) ~= "number" or v < 0 then
                return nil, 'invalid "pool_size"'
            end
            pool_size = v
        elseif k == "max_buffer_reuse" then
            if type(v) ~= "number" or v < 0 then
                return nil, 'invalid "max_buffer_reuse"'
            end
            max_buffer_reuse = v
        elseif k == "periodic_flush" then
            if type(v) ~= "number" or v < 0 then
                return nil, 'invalid "periodic_flush"'
            end
            periodic_flush = v
        elseif k == "ssl" then
            if type(v) ~= "boolean" then
                return nil, '"ssl" must be a boolean value'
            end
            ssl = v
        elseif k == "ssl_verify" then
            if type(v) ~= "boolean" then
                return nil, '"ssl_verify" must be a boolean value'
            end
            ssl_verify = v
        elseif k == "sni_host" then
            if type(v) ~= "string" then
                return nil, '"sni_host" must be a string'
            end
            sni_host = v
        end
    end

    if not (host and port) and not path then
        return nil, "no logging server configured. \"host\"/\"port\" or "
                .. "\"path\" is required."
    end


    if (flush_limit >= drop_limit) then
        return nil, "\"flush_limit\" should be < \"drop_limit\""
    end

    flushing = false
    exiting = false
    connecting = false

    connected = false
    retry_connect = 0
    retry_send = 0

    logger_initted = true

    if periodic_flush then
        if debug then
            ngx_log(DEBUG, "periodic flush enabled for every "
                    .. periodic_flush .. " seconds")
        end
        need_periodic_flush = true
        timer_at(periodic_flush, _periodic_flush)
    end

    return logger_initted
end

function _M.log(msg)
    if not logger_initted then
        return nil, "not initialized"
    end

    local bytes

    if type(msg) ~= "string" then
        msg = tostring(msg)
    end

    local msg_len = #msg

    if (debug) then
        ngx.update_time()
        ngx_log(DEBUG, ngx.now(), ":log message length: " .. msg_len)
    end

    -- response of "_flush_buffer" is not checked, because it writes
    -- error buffer
    if (is_exiting()) then
        exiting = true
        _write_buffer(msg, msg_len)
        _flush_buffer()
        if (debug) then
            ngx_log(DEBUG, "Nginx worker is exiting")
        end
        bytes = 0
    elseif (msg_len + buffer_size < flush_limit) then
        _write_buffer(msg, msg_len)
        bytes = msg_len
    elseif (msg_len + buffer_size <= drop_limit) then
        _write_buffer(msg, msg_len)
        _flush_buffer()
        bytes = msg_len
    else
        _flush_buffer()
        if (debug) then
            ngx_log(DEBUG, "logger buffer is full, this log message will be "
                    .. "dropped")
        end
        bytes = 0
        --- this log message doesn't fit in buffer, drop it
    end

    if last_error then
        local err = last_error
        last_error = nil
        return bytes, err
    end

    return bytes
end

function _M.initted()
    return logger_initted
end

_M.flush = _flush

return _M
```

## init.lua

```
require 'config'
local match = string.match
--ngx_lua如果是0.9.2以上版本，建议正则过滤函数改为ngx.re.find，匹配效率会提高三倍左右。
--因nginx和lua一起的关系，正则表达式使用\d\w\s会出问题，
--local ngxmatch=ngx.re.match
local ngxmatch=ngx.re.find
local unescape=ngx.unescape_uri
local get_headers = ngx.req.get_headers
local optionIsOn = function (options) return options == "on" and true or false end
loghack=optionIsOn(loghack)
--载入socket.lua用于发送log到独立syslog服务器。
local logger = require "socket"
if loghack then


	if not logger.initted() then
          local ok, err = logger.init{
              --host = '192.168.0.1',
              host = 'logserver.local',
              port = 514,
              sock_type = "udp", --udp协议
              flush_limit = 1,	--立即发送
              --drop_limit = 5678,
              pool_size = 100,--连接池大小
          }
          if not ok then
              ngx.log(ngx.ERR, "failed to initialize the logger: ",
                      err)
              return
          end
 end
end


logpath = logdir
rulepath = RulePath
logtofile = optionIsOn(logtofile)
logtoserver = optionIsOn(logtoserver)
UrlDeny = optionIsOn(UrlDeny)
PostCheck = optionIsOn(postMatch)
CookieCheck = optionIsOn(cookieMatch)
WhiteCheck = optionIsOn(whiteModule)
PathInfoFix = optionIsOn(PathInfoFix)
attacklog = optionIsOn(attacklog)
hackipdeny = optionIsOn(hackipdeny)
CCDeny = optionIsOn(CCDeny)
Redirect=optionIsOn(Redirect)
local file = io.open('config')

function getClientIp()
        IP  = ngx.var.remote_addr
        if IP == nil then
                IP  = "unknown"
        end
        return IP
end
function write(logfile,msg)
    local fd = io.open(logfile,"ab")
    if fd == nil then return end
    fd:write(msg)
    fd:flush()
    fd:close()
end

function swrite(msg)
      --保存警告等级要高于nginx error_log的默认等级。
			ngx.log(ngx.CRIT,msg)



end

function log(method,url,data,ruletag)
    if attacklog then
        local realIp = getClientIp()
        local ua = ngx.var.http_user_agent
        if ua == nil then
        	ua="null"
        end
        local servername=ngx.var.host
        local time=ngx.localtime()
        if logtofile then
        local filename = logpath..'/'..servername.."_"..ngx.today().."_sec.log"
        line=realIp.." ["..time.."]".."\""..method.." "..servername..url.."\""..data.."\""..ua.."\""..ruletag.."\"".."\n"
        write(filename,line)
        end
        if logtoserver then
        line=realIp.."\""..method.." "..servername..url.."\""..data.."\""..ua.."\""..ruletag.."\""
        --line="lua_waf:"..line
        swrite(line)
        end
    --发送ip到独立syslog服务器。
    if loghack then  local bytes, err = logger.log(getClientIp()) end
		--只要log记录，说明被攻击，利用denyhackip将ip记录。
		if hackipdeny then  denyhackip(0) end
    end
end
------------------------------------规则读取函数-------------------------------------------------------------------
function read_rule(var)
    file = io.open(rulepath..'/'..var,"r")
    if file==nil then
        return
    end
    t = {}
    for line in file:lines() do
        table.insert(t,line)
    end
    file:close()
    return(t)
end

urlrules=read_rule('url')
argsrules=read_rule('args')
uarules=read_rule('user-agent')
wturlrules=read_rule('whiteurl')
postrules=read_rule('post')
ckrules=read_rule('cookie')


function say_html()
    if Redirect then
        ngx.header.content_type = "text/html"
        ngx.status = ngx.HTTP_FORBIDDEN
        ngx.say(html)
        ngx.exit(ngx.status)
    end
end

function whiteurl()
    if WhiteCheck then
        if wturlrules ~=nil then
						local urlpath = string.gsub(ngx.var.request_uri, "?.*", "")
            for _,rule in pairs(wturlrules) do
            --针对site:开始的进行域名匹配。增加白名单用处。
	            local sitemod,_=string.find(rule,"site:")
	        		if sitemod==1 then
	        			rule=string.gsub(rule,"site:","",1)
	        			--调试whiteurl
	        			--if ngx.var.host=='domino.cqhrss.gov.cn' then
	        			--	log('debug',ngx.var.uri,"",rule)
	        			--end
	        			if ngxmatch(ngx.var.host..ngx.var.uri,rule,"isjo") then
                    return true
                end
	        		else
            		if ngxmatch(ngx.var.uri,rule,"isjo") then
                    return true
								else
									local pathmod,_=string.find(rule,"path:")
			        		if pathmod==1 then
			        			rule=string.gsub(rule,"path:","",1)
			        			if urlpath==rule then
		                    return true
		                end
									end
                end
            	end
            end
        end
    end
    return false
end

function fileExtCheck(ext)
    local items = Set(black_fileExt)
    ext=string.lower(ext)
    if ext then
        for rule,_ in pairs(items) do
            if ngxmatch(ext,rule,"isjo") then
            log('POST',ngx.var.request_uri,"-","file attack with ext "..ext)
            say_html()
            end
        end
    end
    return false
end
function Set (list)
  local set = {}
  for _, l in ipairs(list) do set[l] = true end
  return set
end
function args()
    for _,rule in pairs(argsrules) do
        local args = ngx.req.get_uri_args()
        for key, val in pairs(args) do
            if type(val)=='table' then
                 local t={}
                 for k,v in pairs(val) do
                    if v == true then
                        v=""
                    end
                    table.insert(t,v)
                end
                data=table.concat(t, " ")
            else
                data=val
            end
            if data and type(data) ~= "boolean" and rule ~="" and ngxmatch(unescape(data),rule,"isjo") then
                log('GET',ngx.var.request_uri,"-",rule)
                say_html()
                return true
            end
        end
    end
    return false
end


function url()
    if UrlDeny then
        for _,rule in pairs(urlrules) do
            if rule ~="" and ngxmatch(ngx.var.request_uri,rule,"isjo") then
                log('GET',ngx.var.request_uri,"-",rule)
                say_html()
                return true
            end
        end
    end
    return false
end

function ua()
    local ua = ngx.var.http_user_agent
    if ua ~= nil then
        for _,rule in pairs(uarules) do
            if rule ~="" and ngxmatch(ua,rule,"isjo") then
                log('UA',ngx.var.request_uri,"-",rule)
                say_html()
            return true
            end
        end
    end
    return false
end
function body(data)
    for _,rule in pairs(postrules) do
        if rule ~="" and data~="" and ngxmatch(unescape(data),rule,"isjo") then
            log('POST',ngx.var.request_uri,data,rule)
            say_html()
            return true
        end
    end
    return false
end
function cookie()
    local ck = ngx.var.http_cookie
    if CookieCheck and ck then
        for _,rule in pairs(ckrules) do
            if rule ~="" and ngxmatch(ck,rule,"isjo") then
                log('Cookie',ngx.var.request_uri,"-",rule)
                say_html()
            return true
            end
        end
    end
    return false
end

function denycc()
    if CCDeny then
        local uri=ngx.var.uri
      	local m, err = ngx.re.match(CCrate,'([0-9]+)/([0-9]+)/([0-9]+)')
      	local CCcount=tonumber(m[1]) --计数器上限
        local CCseconds=tonumber(m[2]) --计时器
        local CClimits=tonumber(m[3]) --阻止访问时间
        local token = getClientIp()..uri
        local limit = ngx.shared.limit
        local req,_=limit:get(token) --计数器当前值

        if req then
            if req > CCcount then
                ngx.exit(404)
                return true
            else
            		if req == CCcount then    limit:set(token,CCcount+1,CClimits)  end

                limit:incr(token,1)
								--调试在syslog日志中查看
								--swrite('计数器:'..token..'当前计数器'..req..'阻止访问时间:'..CClimits)

            end
        else
            limit:set(token,1,CCseconds)
        end
    end
    return false
end

--chk为1表示检测值，不增加，不创建，返回检测结果。
function denyhackip(chk)
    if hackipdeny then

       local m, err = ngx.re.match(hackrate,'([0-9]+)/([0-9]+)/([0-9]+)')
      	local hicount=tonumber(m[1]) --计数器上限
        local hiseconds=tonumber(m[2]) --计时器
        local hilimits=tonumber(m[3]) --阻止访问时间
        local token = "hackip"..getClientIp()
        local limit = ngx.shared.limit
        local req,_=limit:get(token) --计数器当前值
        if req then
            if req > hicount then
                ngx.exit(404)
                return true
            else

            		if req == hicount then
            				limit:set(token,hicount+1,hilimits)
            				swrite("ip:"..getClientIp().."因攻击被暂停访问"..hilimits.."秒。")
            				end
                 if chk ~=1 then limit:incr(token,1)      end
                --调试在syslog日志中查看
                --swrite("计数器:"..token.."检测状态:"..chk.."当前计数器"..req.."阻止访问时间:"..hilimits)


            end
        else
        		if chk ~=1 then limit:set(token,1,hiseconds) 	end

        end
    end
    return false
end

function get_boundary()
    local header = get_headers()["content-type"]
    if not header then
        return nil
    end

    if type(header) == "table" then
        header = header[1]
    end

    local m = match(header, ";%s*boundary=\"([^\"]+)\"")
    if m then
        return m
    end

    return match(header, ";%s*boundary=([^\",;]+)")
end

function whiteip()
    if next(ipWhitelist) ~= nil then
        for _,ip in pairs(ipWhitelist) do
            if getClientIp()==ip then
                return true
            end
        end
    end
        return false
end

function blockip()
     if next(ipBlocklist) ~= nil then
         for _,ip in pairs(ipBlocklist) do
             if getClientIp()==ip then
                 ngx.exit(403)
                 return true
             end
         end
     end
         return false
end
```

## config.lua

```
RulePath = "/usr/local/nginx/conf/waf/wafconf/"
attacklog = "on"
--保存日志到文件
logtofile = "on"
logdir = "/site/wwwlogs/waf/"
--保存日志到syslog,采用nginx设置
logtoserver = "off"
--通过syslog日志方式提交hack_ip记录到日志服务器
loghack="off"
UrlDeny="on"
Redirect="on"
CookieMatch="on"
postMatch="on"
whiteModule="on"
black_fileExt={"php","jsp"}
ipWhitelist={"127.0.0.1","192.168.2.1"}
ipBlocklist={"1.0.0.1"}
--违规ip登记，是否限制访问。
--hackrate超过10次/5秒,限制访问1800秒。
hackipdeny="on"
hackrate="10/60/1800"
--cc攻击防范
CCDeny="on"
CCrate="30/60/30"
html=[[
<html><head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no">
<title>嘿，搞啥呢</title>
<style>
p {line-height:20px;}
ul{ list-style-type:none;}
li{ list-style-type:none;}
body{padding:0; margin:0; font:14px/1.5 Microsoft Yahei, 宋体,sans-serif; color:#555;}
</style>
</head>
<body>
 <div style="margin: 0 auto; max-width:1000px; padding-top:70px; overflow:hidden;">
  <div style="padding:0 15px;">
    <div style=" height:40px; line-height:40px; color:#fff; font-size:16px; overflow:hidden; background:#6bb3f6; padding-left:20px;">咳咳，说一下</div>
    <div style="border:1px dashed #cdcece; border-top:none; font-size:14px; background:#fff; color:#555; line-height:24px; height:220px; padding:20px 20px 0 20px; overflow-y:auto;background:#f3f7f9;">
      <p style=" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;"><span style=" font-weight:600; color:#fc4f03;">你被我们抓住了！</span></p>
<p style=" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;">可能原因：人品有问题</p>
<p style=" margin-top:12px; margin-bottom:12px; margin-left:0px; margin-right:0px; -qt-block-indent:1; text-indent:0px;">如何解决：</p>
<ul style="margin-top: 0px; margin-bottom: 0px; margin-left: 0px; margin-right: 0px; -qt-list-indent: 1;"><li style=" margin-top:12px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;">1）给我们发邮件说明；</li>
<li style=" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;">2）给我们打电话说明；</li>
<li style=" margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;">3）什么，上面都不行？转给我们一个亿，人品马上日日高升；</li></ul>
    </div>
  </div>
</div>
</body></html>
]]
```

## whiteurl

```
^/zhaonvpengyou
site:^lianxiwo\.com/
path:/wogeiniliuyi.html
```

## 后话
s
本来还想慢慢说明一下，想想还是算了，这篇文章多是为了自己记住！我主要是基于https://github.com/loveshell/ngx_lua_waf/pull/136修改，然后依虎画猫，增加了一个判断request_uri的小功能！