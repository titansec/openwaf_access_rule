Name
====

access_rule 是 [openwaf](https://github.com/titansec/openwaf) 项目的接入规则模块

通过分析请求的域名和路径等信息，将请求转发至指定的后端服务器

Table of Contents
=================
* [Name](#name)
* [Synopsis](#synopsis)
* [TODO](#todo)
* [Modules Configuration Directives](#modules-configuration-directives)
* [Nginx Variables](#nginx-variables)

Synopsis
========
```txt
#nginx.conf

upstream server_1 {
    server 0.0.0.1; #just an invalid address as a place holder
    balancer_by_lua_file twaf_balancer.lua;
}

server {
    listen 443 ssl;
    server_name _;
    
    set $twaf_https 1;
    set $twaf_upstream_server "";
    
    ssl_certificate nginx.crt;
    ssl_certificate_key nginx.key;
    ssl_certificate_by_lua_file twaf_ssl_cert.lua;
    
    location / {
        proxy_pass $twaf_upstream_server;
    }
}

server {
    listen 80;
    server_name _;
    
    set $twaf_upstream_server "";
    
    location / {
        proxy_pass $twaf_upstream_server;
    }
}

#Configuration
{
    "twaf_access_rule": [
        "rules": [
            {
                "ngx_ssl": true,
                "ngx_ssl_cert": "ngx_a.crt",
                "ngx_ssl_key": "ngx_a.key",
                "host": "test_a.com", 
                "path": "/", 
                "server_ssl": false,
                "forward": "server_1", 
                "forward_addr": "1.1.1.1",
                "forward_port": "8080",
                "uuid": "access_567b067ff2060",
                "policy": "policy_1"
            },
            {   
                "host": "test_c.com", 
                "path": "/", 
                "server_ssl": true,
                "forward": "server_1", 
                "forward_addr": "1.1.1.3",
                "uuid": "access_567b067ff2062",
                "policy": "policy_default"
            }
        ]
    }
}
```
[Back to TOC](#table-of-contents)

TODO
====
* proxy ssl 客户端认证
* proxy 支持域名

[Back to TOC](#table-of-contents)

Modules Configuration Directives
================================

```txt
{
    "twaf_access_rule": [
        "rules": [                                 -- 注意先后顺序
            {                                      
                "ngx_ssl": false,                  -- nginx 认证的开关
                "ngx_ssl_cert": "path",            -- nginx 认证所需 PEM 证书地址
                "ngx_ssl_key": "path",             -- nginx 认证所需 PEM 私钥地址
                "host": "^1\\.1\\.1\\.1$",         -- 域名，正则匹配
                "path": "\/",                      -- 路径，正则匹配
                "port": 80,                        -- 端口，默认 80
                "server_ssl": false,               -- 后端服务器 ssl 开关
                "forward": "server_5",             -- 后端服务器 upstream 名称
                "forward_addr": "1.1.1.2",         -- 后端服务器 ip 地址
                "forward_port": "8080",            -- 后端服务器端口号（缺省80）
                "uuid": "access_567b067ff2060",    -- 用来标记此规则的 uuid
                "policy": "policy_uuid"            -- 安全策略 ID
            }
        ]
    }
}
```
### rules
**syntax:** *"rules": table*

**default:** *none*

**context:** *twaf_access_rule*

table 类型，接入规则，顺序匹配

### ngx_ssl
**syntax:** *"ngx_ssl": true|false*

**default:** *false*

**context:** *twaf_access_rule*

boolean 类型，服务器端(nginx)认证开关，与 client_ssl 组成双向认证，默认关闭

### ngx_ssl_cert
**syntax:** *"ngx_ssl_cert": "path"*

**default:** *none*

**context:** *twaf_access_rule*

string 类型，服务器端(nginx)认证所需 PEM 证书地址，目前仅支持绝对地址

### ngx_ssl_key
**syntax:** *"ngx_ssl_key": "path"*

**default:** *none*

**context:** *twaf_access_rule*

string 类型，服务器端(nginx)认证所需 PEM 私钥地址，目前仅支持绝对地址

### host
**syntax:** *"host": "ip|domain name regex"*

**default:** *none*

**context:** *twaf_access_rule*

string 类型，域名，正则匹配

例如:
```
    "host": "^1\\.1\\.1\\.1$"
    "host": "test\\.com"
    "host": "^.*\\.com$"
    "host": "www.baidu.com"
```

### path
**syntax:** *"path": "regex"*

**default:** *none*

**context:** *twaf_access_rule*

string 类型，路径，正则匹配

例如:
```
    "path": "/"
    "path": "/images"
    "path": "/[a|b]test"
```

### port
**syntax:** *"port": number*

**default:** *80*

**context:** *twaf_access_rule*

number 类型，端口，对应 nginx 中 listen 的端口，默认 80

### server_ssl
**syntax:** *"server_ssl": true|false*

**default:** *false*

**context:** *twaf_access_rule*

boolean 类型，OpenWAF 向后端服务器连接的 ssl 开关

例如:
```
    upstream test {
    	server 1.1.1.1;
    }
    
    http {
    	server {
    	    listen 80;
    	    server_name _;
    	    
    	    location / {
    	        # server_ssl 为 true，相当于 proxy_pass 后为 https
    	    	proxy_pass https://test;
    	        # server_ssl 为 false，相当于 proxy_pass 后为 http
    	    	# proxy_pass http://test;
    	    }
    	}
    }
```

### forward
**syntax:** *"forward": "upstream_uuid"*

**default:** *none*

**context:** *twaf_access_rule*

string 类型，forward 表示后端服务器的 uuid，即 upstream 的名称

```
    #如：forward 值为 test
    upstream test {
        server 1.1.1.1;
    }
```

### forward_addr
**syntax:** *"forward_addr": "ip"*

**default:** *none*

**context:** *twaf_access_rule*

string 类型，forward_addr 表示后端服务器的 ip 地址（TODO：支持域名）

```
    upstream test {
        #如：forward_addr 值为 1.1.1.1
    	server 1.1.1.1;
    }
```

### forward_port
**syntax:** *"forward_port": port*

**default:** *80*

**context:** *twaf_access_rule*

number 类型，forward_port 表示后端服务器端口号，默认 80

```
    upstream test {
    	#如：forward_port 值为 50001
    	server 1.1.1.1:50001;
    }
```

### uuid
**syntax:** *"uuid": "string"*

**default:** *none*

**context:** *twaf_access_rule*

string 类型，接入规则的唯一标识

### policy
**syntax:** *"policy": "policy_uuid"*

**default:** *none*

**context:** *twaf_access_rule*

string 类型，满足此接入规则的请求，所使用安全策略的 uuid

[Back to TOC](#table-of-contents)

Nginx Variables
===============

### $twaf_https
**syntax:** *set $twaf_https 0|1*

**default:** *0*

**context:** *server*

用于标记请求是否通过 ssl 加密

"set $twaf_https 1"，则表示请求通过ssl加密

"set $twaf_https 1"，则表示请求未通过ssl加密

```
server {
    listen 443 ssl;
    set $twaf_https 1;
    ...
}

server {
    listen 80;
    set $twaf_https 0;
    ...
}
```

### $twaf_upstream_server
**syntax:** *set $twaf_upstream_server ""*

**default:** *none*

**context:** *server*

用于指定后端服务器地址，只需初始化为空字符串即可，其值由 "server_ssl" 和 "forward" 确定

```
upstream server_1 {
    ...
}

upstream server_2 {
    ...
}

server {
    ...
    
    set $twaf_upstream_server "";
    location / {
        ...
        proxy_pass $twaf_upstream_server;
    }
}

若 "server_ssl" 值为 true, "forward" 值为 "server_1"
等价于proxy_pass https://server_1;

若 "server_ssl" 值为 false, "forward" 值为 "server_2"
等价于 proxy_pass http://server_2;
```

[Back to TOC](#table-of-contents)
