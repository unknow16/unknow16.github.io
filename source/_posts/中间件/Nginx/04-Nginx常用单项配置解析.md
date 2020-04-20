---
title: 04-Nginx常用单项配置解析
toc: true
date: 2019-07-15 15:31:58
tags:
categories:
---





## location块

语法规则： location [=|~|~*|^~]     /uri/       { … }



- = 表示精确匹配,这个优先级也是最高的
- ~  表示区分大小写的正则匹配
- ~* 表示不区分大小写的正则匹配
- ^~ 表示uri以某个常规字符串开头
- !~ 区分大小写不匹配的正则
- !~* 不区分大小写不匹配的正则
- / 通用匹配，任何请求都会匹配到，默认匹配



优先级： 首先匹配 =，其次匹配^~, 其次是按文件中顺序的正则匹配，最后是交给 / 通用匹配

```
location / {   
	echo "/"; //需要安装echo模块才行,这边大家可以改成各自的规则
}
location = / {   
	echo "=/";
}
location = /nginx {   
	echo "=/nginx";
}
location ~ \.(gif\|jpg\|png\|js\|css)$ {    // 区分大小写，以gif、jpg等结尾的url
	echo "small-gif/jpg/png";
}
location ~* \.png$ {    // 不区分大小写，以png结尾的url,如果xxx.PNG也能匹配
	echo "all-png";
}
location ^~ /static/ {    // 以static开头的url
	echo "static";
}
```



## root&alias文件路径配置

Nginx指定文件路径有两种方式root和alias，root与alias主要区别在于nginx如何解释location后面的uri，这会使两者分别以不同的方式将请求映射到服务器文件上。

```
[root]
语法：root path 
默认值：root html
可用于配置段中：http、server、location、if  

[alias]
语法：alias path
可用于配置段中：location  
```

- 实例

```
location ^~ /weblogs/ {		
    root /data/weblogs/www.yfming.com;		
    autoindex on;		
    auth_basic            "Restricted";		
    auth_basic_user_file  passwd/weblogs;
}
```

root会把其指定的路径和请求的路径联合起来。

如果一个请求的URI是/weblogs/httplogs/www.yfming.com-access.log时，web服务器将会返回服务器上的/data/weblogs/www.yfming.com/weblogs/httplogs/www.yfming.com-access.log的文件。

```
location ^~ /binapp/ {  		
    limit_conn limit 4;		
    limit_rate 200k;		
    internal;  		
    alias /data/statics/bin/apps/;
}
```

alias会把location后面配置的路径丢弃掉，把当前匹配到的目录指向到指定的目录。

如果一个请求的URI是/binapp/a.yfming.com/favicon时，web服务器将会返回服务器上的/data/statics/bin/apps/a.yfming.com/favicon.jgp的文件。

- alias注意点

1.  使用alias时，目录名后面一定要加"/"。
2.  alias可以指定任何名称。
3.  alias在使用正则匹配时，必须捕捉要匹配的内容并在指定的内容处使用。
4.  alias只能位于location块中。



## 参考资料
> - []()
> - []()
