---
title: 05-Nginx日志相关配置
toc: true
date: 2019-07-15 15:34:58
tags:
categories:
---



## access_log访问日志配置

```
语法格式1:   access_log path [format [buffer=size] [gzip[=level]] [flush=time] [if=condition]];
语法格式1:   access_log off;  # 即关闭日志
默认值   : access_log logs/access.log combined;
作用域   : http, server, location, if in location, limit_except
```

- 实例

```
access_log  logs/access.log  main
```

表示nginx目录下的logs/access.log保存日志，采用名为main的日志格式



## log_format 日志格式

```
语法格式:   	log_format name [escape=default|json] string ...;
默认值    :    log_format combined "...";
作用域    :    http
```

- 实例

```
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';

log_format logstash_json '{"@timestamp":"$time_iso8601",'
       '"host": "$server_addr",'
       '"client": "$remote_addr",'
       '"size": $body_bytes_sent,'
       '"responsetime": $request_time,'
       '"domain": "$host",'
       '"url":"$request_uri",'
       '"referer": "$http_referer",'
       '"agent": "$http_user_agent",'
       '"status":"$status",'
       '"x_forwarded_for":"$http_x_forwarded_for"}';
```

- 实例main格式输出

```
# 变量值为空时为短中划线
14.107.157.10 - - [15/Jul/2019:14:33:23 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.108 Safari/537.36"
```

- 常见的日志变量
  1. `$remote_addr`, `$http_x_forwarded_for` 记录客户端IP地址
  2. `$remote_user`记录客户端用户名称
  3. `$request`记录请求的URL和HTTP协议(GET,POST,DEL,等)
  4. `$status`记录请求状态
  5. `$body_bytes_sent`发送给客户端的字节数，不包括响应头的大小； 该变量与Apache模块mod_log_config里的“%B”参数兼容。
  6. `$bytes_sent`发送给客户端的总字节数。
  7. `$connection`连接的序列号。
  8. `$connection_requests` 当前通过一个连接获得的请求数量。
  9. `$msec` 日志写入时间。单位为秒，精度是毫秒。
  10. `$pipe`如果请求是通过HTTP流水线(pipelined)发送，pipe值为“p”，否则为“.”。
  11. `$http_referer` 记录从哪个页面链接访问过来的
  12. `$http_user_agent`记录客户端浏览器相关信息
  13. `$request_length`请求的长度（包括请求行，请求头和请求正文）。
  14. `$request_time` 请求处理时间，单位为秒，精度毫秒； 从读入客户端的第一个字节开始，直到把最后一个字符发送给客户端后进行日志写入为止。
  15. `$time_iso8601 ISO8601`标准格式下的本地时间。
  16. `$time_local`通用日志格式下的本地时间。



## error_log错误日志配置

```
语法:  error_log file [debug | info | notice | warn | error | crit | alert | emerg];
默认值: error_log logs/error.log error;
配置段: main, http, server, location
```

默认只记录error级别的错误日志



## 参考资料
> - []()
> - []()
